# Native Preludes Should Reach Lua Through a Handle-Based C API over the Engine's mlua

- Type: analytical / recommendation report, with the full proposed declaration
- Audience: the owner and the engineer or agent who would build the boundary
- Decision served: whether to adopt the declaration in section 2 as version 1.0 of the boundary between the Engine and the native code in packages, and what to build first
- Inputs: the 2026-10-18 design conversation between the owner (Vinnie Falco) and a Cursor agent; the `runvil` code at the workspace head, which pins mlua 0.12.0, mlua-sys 0.11.0, and Lua 5.5.0 from lua-src 550.1.1 (`Cargo.lock`); the `runvil-docs` language guide; and the 2026-10-04 design record "Runvil Plugin and Composition Design"
- Status: proposal; no code exists, and one owner ruling is needed first (section 6.1)
- Vocabulary: revised on 2026-10-18 to the names of the "Plugin Registry and Info Rename" plan. A package is the unit Plugins ship in, a dynamic library or a linked crate, and it can hold several Plugins. A Plugin is named by its fully qualified name (FQN), such as `acme/url`, and installed under a Plugin id, such as `url`. Static data is called "info", as in `PluginInfo`. A native prelude is a Plugin's compiled code that defines globals in each section VM, and "native" means compiled code only; the Engine-built Plugin kind is called built-in

## Executive summary

Adopt the declaration in section 2 as API 1.0. A package exports one function, `runvil_package_init`. The Harness calls it once per process with an `RvHost` table, and the package names itself and registers as many Plugins as it holds. A prompt declares one of those Plugins under `plugins:`, as it declares any Plugin today, and the Plugin's native prelude, its `open` function, then runs in every section VM of that prompt. `open` defines globals by setting fields on the Plugin's prelude environment, the same table a Lua prelude writes to, so the Engine's collision checks and sealing apply unchanged.

All Lua work goes through `RvLua`, a table of 30 functions that the Engine implements with its own mlua. A plugin never sees a `lua_State*`, the Lua stack, the registry, or an mlua type. It holds tagged values instead: nil, booleans, integers, floats, and `json.null` travel in place, while strings, tables, functions, and userdata are handles the Engine checks on every call. Because the Engine's single mlua does all the work, Lua errors stop inside the Engine and come back as status codes, lists use the Engine's own marker, and plugins keep working when mlua or Lua changes version.

The safety choices remove every facility that can break an Engine rule. The API has no raw set, no protected-metatable reads, no globals table, no code loading, no coroutines, no unordered iteration, and no memory addresses. A memory-limit failure or a cancellation is sticky: the plugin cannot clear it, and the Engine raises it when the plugin returns. Contexts and handles are passed by value and validated, so a stale or foreign one returns `RV_ERR_MISUSE` instead of crashing. Userdata belongs to a type the Plugin declares, and the Engine checks that type before every callback.

Plugins keep what they need: building and reading any Lua data, reading `var` and `argv` the way Lua code does, calling Lua callbacks, objects with methods and comparisons, the list marker, sorted iteration, bulk JSON conversion, and cancellation polling. The surface is likely complete for Plugins in the class of `json`, `url`, and `markdown`. Confidence in that is medium: every function was checked against the Engine's existing rules, but no plugin has been built against the declaration. One ruling comes first, because the 2026-10-04 design record says a prelude is pure Lua, and a native `open` function is an exception to that rule (section 6.1).

## Contents

1. [Six criteria judge the boundary](#1-six-criteria-judge-the-boundary)
2. [The declaration: one export, one host table, and 30 Lua functions](#2-the-declaration-one-export-one-host-table-and-30-lua-functions)
3. [Each choice trades raw Lua power for a checked outcome](#3-each-choice-trades-raw-lua-power-for-a-checked-outcome)
4. [The declaration leaves out every facility that can break an Engine rule](#4-the-declaration-leaves-out-every-facility-that-can-break-an-engine-rule)
5. [Three alternatives fail at least one criterion](#5-three-alternatives-fail-at-least-one-criterion)
6. [One owner ruling and four limits remain](#6-one-owner-ruling-and-four-limits-remain)
7. [Next steps: rule, spike, guard, then build the SDK](#7-next-steps-rule-spike-guard-then-build-the-sdk)
8. [References](#references)
9. [Appendix A. A complete one-function package](#appendix-a-a-complete-one-function-package)

## 1. Six criteria judge the boundary

The boundary exists so Plugin code can live in its own workspace, built and tested apart from the Engine. That makes the C declaration the only contract between the two sides, and the compiler no longer checks one against the other. Every criterion below therefore asks one question: when plugin code is wrong, does the mistake surface as an error, or as a crash or a silently wrong run?

1. **Contained errors.** No Lua error unwinds through plugin code, and no plugin failure unwinds through the Engine. Lua reports errors with `longjmp`, which skips C++ destructors and Rust `Drop`.
2. **Engine rules hold.** The sandbox, the `_G` guard, `var` validation, sealed Plugin tables, the instruction hook that makes a run cancellable, the Lua memory limit, and the yield protocol behind effects all survive any sequence of plugin calls.
3. **Deterministic results.** A plugin sees a table's string, number, and boolean keys in the same order on every run, and nothing it can read depends on a memory address, because replay runs native code again.
4. **Detected misuse.** A stale or foreign handle or context, a value of the wrong kind, a NULL pointer, or a malformed struct returns a status instead of causing undefined behavior.
5. **Stable contract.** Plugins keep working across mlua and Lua upgrades, and the declaration grows only by appending.
6. **Sufficient function.** The API supports the Plugins in view, `json`, `url`, and `markdown`, and others of the same kind, such as regular expressions or version parsing.

## 2. The declaration: one export, one host table, and 30 Lua functions

The block below is the complete proposed header, `runvil_lua.h`. It is C; the fence is tagged `lua` at the owner's request. Section 3 explains each choice, and section 4 lists what was left out and why.

```lua
/* runvil_lua.h: the native prelude API, version 1.0.
 *
 * A package exports runvil_package_init. The Harness calls it once per
 * process, passing an RvHost, and the package names itself and registers
 * its Plugins through it. A Plugin's open function, its native prelude,
 * runs in every section VM of a prompt that declares the Plugin under
 * plugins:, and every function it installs works through the RvLua table
 * that RvHost points to.
 *
 * Rules every declaration follows:
 * - Every RvLua function returns an RvStatus. Out-pointers receive nil,
 *   zero, false, or NULL whenever the status is not RV_OK.
 * - Text is always a pointer plus a byte length. Nothing is NUL-terminated,
 *   and Lua strings may contain NUL bytes.
 * - Every struct a plugin passes in begins with its own size, and every
 *   reserved field must be zero.
 * - Kinds, statuses, and flags are uint32_t constants, never C enums, so
 *   their size is the same in every compiler.
 * - No function is variadic.
 * - No plugin function may unwind: no C++ exception and no Rust panic may
 *   leave it.
 */

#ifndef RUNVIL_LUA_H
#define RUNVIL_LUA_H

#include <stdbool.h>
#include <stddef.h>
#include <stdint.h>

#define RV_API_MAJOR 1u
#define RV_API_MINOR 0u

#if defined(_WIN32)
#define RV_CALL __cdecl
#define RV_EXPORT __declspec(dllexport)
#else
#define RV_CALL
#define RV_EXPORT __attribute__((visibility("default")))
#endif

/* A string literal and its byte length, for any text parameter pair. */
#define RV_TEXT(literal) (literal), (sizeof(literal) - 1u)

/* The most values one call passes or returns. */
#define RV_MAX_VALUES 255u

/* The most Lua values one userdata instance holds. */
#define RV_MAX_USER_VALUES 4u


/* Statuses. Every status other than RV_OK leaves a message that
 * error_message reads. */

typedef uint32_t RvStatus;

#define RV_OK         0u /* success */
#define RV_ERR_LUA    1u /* Lua raised an error, which the call now holds.
                            clear_error catches it; returning the status
                            raises it unchanged. */
#define RV_ERR_FATAL  2u /* the VM reached its memory limit or the run was
                            cancelled. It cannot be cleared, every later
                            RvLua call with the same context returns it, it
                            reaches every enclosing plugin call, and the
                            Engine raises it when the plugin function
                            returns, whatever the function returns. */
#define RV_ERR_TYPE   3u /* a value had the wrong kind for the call;
                            nothing was raised in Lua */
#define RV_ERR_MISUSE 4u /* the plugin broke this contract: a stale or
                            foreign context or handle, a NULL pointer, a bad
                            size, a nonzero reserved field, an undeclared
                            type, or more than RV_MAX_VALUES values */


/* Values. */

typedef uint32_t RvKind;

#define RV_NIL      0u
#define RV_BOOLEAN  1u
#define RV_INTEGER  2u
#define RV_NUMBER   3u /* a float */
#define RV_NULL     4u /* json.null */
#define RV_STRING   5u
#define RV_TABLE    6u
#define RV_FUNCTION 7u
#define RV_USERDATA 8u
#define RV_OPAQUE   9u /* a coroutine or any other value: pass it on,
                          never inspect it */

/* One Lua value. Scalars are held in place. Strings, tables, functions,
 * userdata, and opaque values are handles, which only the Engine creates.
 * A handle stays valid until its call returns or release_to passes it. */
typedef struct RvValue {
    RvKind kind;
    uint32_t reserved; /* zero */
    union {
        bool boolean;    /* RV_BOOLEAN */
        int64_t integer; /* RV_INTEGER */
        double number;   /* RV_NUMBER */
        uint64_t handle; /* RV_STRING through RV_OPAQUE */
    } as;
} RvValue;

static inline RvValue rv_nil(void) {
    RvValue v = {RV_NIL, 0u, {0}};
    return v;
}

static inline RvValue rv_null(void) {
    RvValue v = {RV_NULL, 0u, {0}};
    return v;
}

static inline RvValue rv_boolean(bool b) {
    RvValue v = {RV_BOOLEAN, 0u, {0}};
    v.as.boolean = b;
    return v;
}

static inline RvValue rv_integer(int64_t i) {
    RvValue v = {RV_INTEGER, 0u, {0}};
    v.as.integer = i;
    return v;
}

static inline RvValue rv_number(double n) {
    RvValue v = {RV_NUMBER, 0u, {0}};
    v.as.number = n;
    return v;
}

/* The type name the Engine's own messages use for a kind, as in
 * "expects a string, got integer". */
static inline const char *rv_kind_name(RvKind kind) {
    switch (kind) {
    case RV_NIL: return "nil";
    case RV_BOOLEAN: return "boolean";
    case RV_INTEGER: return "integer";
    case RV_NUMBER: return "number";
    case RV_NULL: return "lightuserdata";
    case RV_STRING: return "string";
    case RV_TABLE: return "table";
    case RV_FUNCTION: return "function";
    case RV_USERDATA: return "userdata";
    default: return "other";
    }
}


/* Contexts and other call-scoped tokens. All are passed by value and
 * checked by the Engine on every use. */

/* One plugin call: one run of open, or one call of a plugin function.
 * Valid only while that call runs, on its own thread, and only as the
 * innermost active call on that thread. */
typedef struct RvCx { uint64_t bits; } RvCx;

/* A position in a call's handle arena. */
typedef struct RvMark { uint64_t bits; } RvMark;

/* A table walk owned by one call. */
typedef struct RvIter { uint64_t bits; } RvIter;

/* The package being loaded. Valid only during runvil_package_init. */
typedef struct RvPackage { uint64_t bits; } RvPackage;


/* Plugin callbacks. */

/* A plugin function. args holds nargs values, valid for this call only.
 * Results go through return_values; returning without calling it returns
 * no values. Return RV_OK, or the non-OK status of an RvLua call to raise
 * the error that call holds. */
typedef RvStatus(RV_CALL *RvFunction)(RvCx cx, void *data,
                                      const RvValue *args, size_t nargs);

/* Frees plugin data. It runs once, when Lua collects the owner or the VM
 * closes. It receives no context and may not call RvLua. */
typedef void(RV_CALL *RvDrop)(void *data);

/* A Plugin's native prelude, run once in each section VM of a prompt that
 * declares the Plugin. env is the Plugin's prelude environment: every
 * field set on it becomes a global once open returns. plugin_id is the
 * Plugin id the Host installed the Plugin under. */
typedef RvStatus(RV_CALL *RvOpen)(RvCx cx, void *plugin_data, RvValue env,
                                  const char *plugin_id,
                                  size_t plugin_id_len);


/* Info structs. */

/* new_table flag: mark the new table as a list, as json.array() does. */
#define RV_TABLE_LIST 1u

/* A plugin function. The Engine copies it, so it need not outlive the
 * call to new_function. */
typedef struct RvFunctionInfo {
    uint32_t size;     /* sizeof(RvFunctionInfo) */
    uint32_t reserved; /* zero */
    const char *name;  /* such as "url.parse", for messages */
    size_t name_len;
    RvFunction call;
    RvDrop drop;       /* NULL when data needs no freeing */
} RvFunctionInfo;

/* One method of a userdata type, called as u:name(...). */
typedef struct RvMethod {
    const char *name;
    size_t name_len;
    RvFunction call;   /* data is the instance's data; args[0] is the instance */
} RvMethod;

/* A userdata type. List every type in RvPluginInfo.types; the type
 * must stay valid for the life of the process. The Engine builds one
 * protected metatable per type and VM: getmetatable returns the type name,
 * __gc calls drop, and the plugin never sees the metatable. Before every
 * callback below, the Engine checks that an operand is an instance of this
 * type and raises otherwise, so data always belongs to this type. For eq,
 * lt, and le, data is the left operand's when both are instances. */
typedef struct RvUserdataType {
    uint32_t size;           /* sizeof(RvUserdataType) */
    uint32_t user_values;    /* Lua values each instance holds,
                                0 to RV_MAX_USER_VALUES */
    const char *name;        /* such as "url.Url" */
    size_t name_len;
    const RvMethod *methods;
    size_t method_count;
    RvFunction index;        /* optional: u.key not among the methods;
                                args (u, key) */
    RvFunction newindex;     /* optional: u.key = v; args (u, key, v).
                                Absent, the assignment raises. */
    RvFunction tostring;     /* optional. Absent, tostring(u) is the type
                                name, never an address. */
    RvFunction eq;           /* optional: u == v, both userdata. Absent,
                                identity. */
    RvFunction lt;           /* optional: u < v */
    RvFunction le;           /* optional: u <= v */
    RvFunction len;          /* optional: #u */
    RvFunction call;         /* optional: u(...); args[0] is u */
    RvDrop drop;             /* required: frees an instance's data */
} RvUserdataType;

/* The package itself. Its name and version, with the dynamic library's
 * hash, are recorded in each run's Plugin roster for every Plugin it
 * provides, so a replay against a different build is caught. */
typedef struct RvPackageInfo {
    uint32_t size;           /* sizeof(RvPackageInfo) */
    uint32_t reserved;       /* zero */
    const char *name;        /* such as "acme-url" */
    size_t name_len;
    const char *version;     /* such as "0.1.0" */
    size_t version_len;
} RvPackageInfo;

/* One Plugin the package provides. The Harness copies its text and checks
 * every field, the types included, before register_plugin returns. */
typedef struct RvPluginInfo {
    uint32_t size;           /* sizeof(RvPluginInfo) */
    uint32_t reserved;       /* zero */
    const char *fqn;         /* the fully qualified name, "vendor/name"; the
                                Host installs the Plugin under the second
                                segment unless it picks another Plugin id */
    size_t fqn_len;
    const char *description; /* one line, for a Host listing Plugins */
    size_t description_len;
    RvOpen open;             /* the native prelude; NULL for a Plugin with
                                only a Lua prelude */
    const char *lua_prelude; /* optional Lua, run after open in the same
                                environment */
    size_t lua_prelude_len;
    const RvUserdataType *const *types; /* every type the Plugin creates */
    size_t type_count;
    void *plugin_data;       /* passed to open in every VM, on any thread;
                                what it points to must be thread-safe */
} RvPluginInfo;


/* The Lua API. The Engine implements every entry with its own mlua. A
 * package built against minor version N reads only the entries version N
 * declares; later versions only append. */

typedef struct RvLua RvLua;

struct RvLua {
    uint32_t size;           /* sizeof(RvLua) as the Engine built it */
    uint32_t reserved;

    /* Errors and cancellation */

    /* Holds an author error with this message and returns RV_ERR_LUA.
     * Returning that status raises it; Lua's pcall reports kind lua. */
    RvStatus(RV_CALL *raise)(RvCx cx, const char *message,
                             size_t message_len);

    /* The held status's message, valid until clear_error or the end of
     * the call. */
    RvStatus(RV_CALL *error_message)(RvCx cx, const char **message,
                                     size_t *message_len);

    /* Drops a held RV_ERR_LUA, RV_ERR_TYPE, or RV_ERR_MISUSE. Returns
     * RV_ERR_FATAL instead when that is held. */
    RvStatus(RV_CALL *clear_error)(RvCx cx);

    /* RV_ERR_FATAL once the run is cancelled. Every RvLua call checks
     * too; call this in long loops that make no other call. */
    RvStatus(RV_CALL *check_cancel)(RvCx cx);

    /* Handle arena */

    RvStatus(RV_CALL *mark)(RvCx cx, RvMark *out);

    /* Releases every handle created after mark. A released handle is
     * refused afterward, even when its slot is reused. */
    RvStatus(RV_CALL *release_to)(RvCx cx, RvMark mark);

    /* Strings */

    /* Copies len bytes into a new Lua string. */
    RvStatus(RV_CALL *new_string)(RvCx cx, const char *bytes, size_t len,
                                  RvValue *out);

    /* Borrows a string's bytes, valid while its handle is. RV_STRING
     * only: numbers are never converted. */
    RvStatus(RV_CALL *string_bytes)(RvCx cx, RvValue string,
                                    const char **bytes, size_t *len);

    /* Tables */

    /* A new table. The hints only presize it; flags may hold
     * RV_TABLE_LIST. */
    RvStatus(RV_CALL *new_table)(RvCx cx, uint32_t list_hint,
                                 uint32_t field_hint, uint32_t flags,
                                 RvValue *out);

    /* Marks a table as a list with json.array(t)'s rules: a table with
     * any other metatable is refused. */
    RvStatus(RV_CALL *mark_list)(RvCx cx, RvValue table);

    /* True only for a table that has the list marker. */
    RvStatus(RV_CALL *is_list)(RvCx cx, RvValue value, bool *out);

    /* object[key], running __index as Lua would. object is a table or a
     * userdata, so var tables, argv, and sealed Plugin tables read as they
     * do in Lua code. */
    RvStatus(RV_CALL *get)(RvCx cx, RvValue object, RvValue key,
                           RvValue *out);
    RvStatus(RV_CALL *get_field)(RvCx cx, RvValue object, const char *name,
                                 size_t name_len, RvValue *out);

    /* table[key] without metamethods. No Lua code runs. */
    RvStatus(RV_CALL *raw_get)(RvCx cx, RvValue table, RvValue key,
                               RvValue *out);

    /* object[key] = value, running __newindex as Lua would, so var
     * validation and every read-only refusal apply. There is no raw set. */
    RvStatus(RV_CALL *set)(RvCx cx, RvValue object, RvValue key,
                           RvValue value);
    RvStatus(RV_CALL *set_field)(RvCx cx, RvValue object, const char *name,
                                 size_t name_len, RvValue value);

    /* A table's border length, without __len. */
    RvStatus(RV_CALL *raw_len)(RvCx cx, RvValue table, int64_t *out);

    /* setmetatable(table, metatable) with Lua's protection rules, and a
     * refusal of any metatable holding __gc, __mode, or __close.
     * metatable is a table or nil. */
    RvStatus(RV_CALL *set_metatable)(RvCx cx, RvValue table,
                                     RvValue metatable);

    /* rawequal(a, b): identity for handles, value for scalars. Never runs
     * __eq. */
    RvStatus(RV_CALL *raw_equal)(RvCx cx, RvValue a, RvValue b, bool *out);

    /* Iteration */

    /* Captures a table's keys once, in the order pairs uses: the list
     * part, then booleans, numbers, and strings sorted, then other keys.
     * No metamethod runs, so sealed views such as var tables walk as
     * empty; read those with get or to_json. */
    RvStatus(RV_CALL *iter_begin)(RvCx cx, RvValue table, RvIter *out);

    /* The next live entry. A key cleared since iter_begin is skipped. At
     * the end, *done is true and key and value are nil. */
    RvStatus(RV_CALL *iter_next)(RvCx cx, RvIter iter, RvValue *key,
                                 RvValue *value, bool *done);

    /* Functions */

    /* A Lua function backed by info->call. data reaches every call and is
     * freed with info->drop when Lua collects the function. On failure
     * the Engine calls info->drop(data) before returning, so ownership of
     * data always passes to the Engine. */
    RvStatus(RV_CALL *new_function)(RvCx cx, const RvFunctionInfo *info,
                                    void *data, RvValue *out);

    /* callee(args...), keeping nresults results and filling missing ones
     * with nil. callee is a function or a value with __call. A yield
     * inside fails as RV_ERR_LUA, and a jump inside is still recorded, as
     * under pcall. */
    RvStatus(RV_CALL *call)(RvCx cx, RvValue callee, const RvValue *args,
                            size_t nargs, RvValue *results,
                            size_t nresults);

    /* Sets the current plugin function's results; a second call replaces
     * the first. Refused inside open, which returns no values. */
    RvStatus(RV_CALL *return_values)(RvCx cx, const RvValue *values,
                                     size_t count);

    /* Userdata */

    /* A new instance of a type the Plugin declared. The Engine owns data
     * from here on, also on failure, and frees it with type->drop. */
    RvStatus(RV_CALL *new_userdata)(RvCx cx, const RvUserdataType *type,
                                    void *data, RvValue *out);

    /* The data of an instance of exactly this type, valid while the
     * handle is. Any other value returns RV_ERR_TYPE. */
    RvStatus(RV_CALL *userdata_data)(RvCx cx, RvValue value,
                                     const RvUserdataType *type,
                                     void **out);

    /* The Lua values an instance holds, by index below type->user_values.
     * Lua's collector traces them, so a cycle through them is still
     * freed. */
    RvStatus(RV_CALL *get_user_value)(RvCx cx, RvValue value,
                                      const RvUserdataType *type,
                                      uint32_t index, RvValue *out);
    RvStatus(RV_CALL *set_user_value)(RvCx cx, RvValue value,
                                      const RvUserdataType *type,
                                      uint32_t index, RvValue item);

    /* JSON, through the Engine's own conversions */

    /* Parses JSON as json.decode does: lists marked, null as json.null, at
     * most 8,388,608 bytes and 127 levels. who starts the error message,
     * as in "url.query: expected value at line 1 column 1". */
    RvStatus(RV_CALL *from_json)(RvCx cx, const char *who, size_t who_len,
                                 const char *json, size_t json_len,
                                 RvValue *out);

    /* Encodes as json.encode does, reading through var, argv, and other
     * sealed views, and returns a string. who replaces json.encode in its
     * messages, as in "url.build: value.query.a must be JSON data, got
     * function". */
    RvStatus(RV_CALL *to_json)(RvCx cx, const char *who, size_t who_len,
                               RvValue value, RvValue *out);
};


/* The host table and the package's one export. */

typedef struct RvHost {
    /* These five members never move in any version, so a package built
     * for any version can always read the versions and refuse. */
    uint32_t size;           /* sizeof(RvHost) as the Harness built it */
    uint32_t api_major;      /* the package requires RV_API_MAJOR */
    uint32_t api_minor;      /* the package requires at least the
                                RV_API_MINOR it was built against */
    uint32_t reserved;

    /* Refuses the whole package with a reason a model can read. Return its
     * status from runvil_package_init; every Plugin in the package is then
     * unavailable with that reason. */
    RvStatus(RV_CALL *refuse)(RvPackage package, const char *reason,
                              size_t reason_len);

    /* Names the package and its version. Call it once, before the first
     * register_plugin. */
    RvStatus(RV_CALL *describe_package)(RvPackage package,
                                        const RvPackageInfo *info);

    /* Registers one Plugin. An FQN that any package already registered is
     * refused. */
    RvStatus(RV_CALL *register_plugin)(RvPackage package,
                                       const RvPluginInfo *plugin);

    const RvLua *lua;        /* valid for the life of the process */
} RvHost;

/* The package's one export, called once per process. A non-OK status makes
 * every Plugin in the package unavailable. The Harness never unloads a
 * dynamic library it loaded. A linked package exports the same function
 * under its own name, and the Host passes it to the Harness directly. */
RV_EXPORT RvStatus RV_CALL runvil_package_init(const RvHost *host,
                                               RvPackage package);

#endif /* RUNVIL_LUA_H */
```

## 3. Each choice trades raw Lua power for a checked outcome

### 3.1 Scalars travel in place, and only the Engine makes handles

`RvValue` is a 16-byte tagged struct. Nil, booleans, integers, floats, and `json.null` sit in the struct itself, so a plugin reads a number argument or builds an integer result without any call. Strings, tables, functions, userdata, and every other value are handles: 64-bit numbers that only the Engine issues. A plugin can write any bits into a handle field, but the Engine refuses a handle it did not issue to that call, so a forged or corrupted handle returns `RV_ERR_MISUSE` (criterion 4).

`json.null` has its own kind, `RV_NULL`, because the Engine and the `json` global treat it as JSON null everywhere. Every other light userdata, and every coroutine, arrives as `RV_OPAQUE`: a plugin can store it in a table or pass it back, but cannot look inside it. `rv_kind_name` returns the type names the Engine's own messages use, such as `integer` and `lightuserdata`, because every Engine argument error names types with mlua's `Value::type_name` (`crates/runvil-internal/lua/src/json.rs`, `array`).

### 3.2 Contexts and handles are checked on every call

A context, `RvCx`, names one plugin call: one run of `open`, or one call of a plugin function. It is passed by value rather than as a pointer, so a plugin cannot dereference freed Engine memory through it. The Engine keeps a stack of active calls for each thread and accepts a context only when it is the innermost one on the calling thread. That turns three mistakes into a status: using a context after its call returned, using it from another thread, and using an outer call's context while a nested call runs.

Each call owns a handle arena, and the Engine stores the arena's values in a Lua table rather than holding one mlua reference per handle. mlua keeps each live reference on an internal Lua stack (`ref_thread`, mlua 0.12.0 `src/state/extra.rs`, line 58), and Lua 5.5 caps any stack at 1,000,000 slots (`LUAI_MAXSTACK`, Lua 5.5.0 `ldo.c`, lines 186-196). A table-backed arena has no such ceiling: live handles cost Lua memory and fall under the VM's memory limit like any other value. `mark` and `release_to` free the handles a loop no longer needs, as JNI's local frames do. Each slot has a generation number, so a released handle stays refused even after its slot is reused.

### 3.3 Lua errors stop inside the Engine and come back as one of four statuses

Every `RvLua` function does its Lua work through mlua, which already runs each operation in a protected call and returns `mlua::Result`. The Engine turns that result into a status before returning to the plugin, so no `longjmp` ever crosses plugin frames (criterion 1). In the other direction, the contract forbids a plugin function to unwind, and the planned Rust SDK enforces that by catching every panic at the boundary.

The catch rules follow the Engine's own `pcall` wherever that is safe. `RV_ERR_LUA` covers everything Lua's `pcall` can catch. A plugin catches it with `clear_error`, or returns the status to raise the held error unchanged, which keeps its kind and message intact. A `jump` made inside a Lua callback reaches the plugin as `RV_ERR_LUA` holding the text `jump transfer`, and catching it does not cancel the jump, exactly as the language guide documents for `pcall` (`runvil-docs`, `src/language/08-jump-and-call.md`, "Catching a jump with `pcall`").

`RV_ERR_FATAL` covers cancellation and the memory limit, and it cannot be cleared. Once it occurs, every later call with that context returns it, and the Engine raises the saved error when the plugin function returns, whatever the function returns. For cancellation this matches the Engine: its normalizing `pcall` raises again any failure caught while the run is cancelled (`crates/runvil-internal/lua/src/__impl_coro.lua`, lines 78-84). For the memory limit it is stricter than Lua, whose `pcall` can catch an out-of-memory error. The difference is deliberate. A Plugin function that answers every failure with "invalid input" would otherwise turn an out-of-memory error into a wrong answer, and no such function needs to survive one.

`RV_ERR_TYPE` and `RV_ERR_MISUSE` raise nothing in Lua. They report a value of the wrong kind and a broken contract, and each leaves a message. A plugin function that returns either one raises that message as an ordinary error, so even a careless plugin produces a readable failure rather than a crash.

### 3.4 Tables keep the Engine's guards because the only write runs `__newindex`

`set` and `set_field` behave like assignment in Lua: they run `__newindex`. That keeps every Engine guard in force. A write to a `var` table passes `var`'s validation, which checks that the value is JSON data and rebuilds it as guarded data (`crates/runvil-internal/lua/src/sys.rs`, lines 197-205). A write to `sys`, to a sealed Plugin table, or to `argv` outside the H1 pass raises the same refusal Lua code gets. The declaration has no raw set, because a raw set is the one write that skips all of those guards. Plugin-built tables lose nothing: a table with no metatable, or with only the list marker, has no `__newindex`, so `set` on it is already a plain store.

Reads come in two forms. `get` runs `__index` like Lua's own indexing, so a plugin reads `var.items` or a sealed Plugin table exactly as Lua code does. `raw_get` runs nothing, for code that must not execute Lua. Neither reveals a protected metatable, and the declaration has no metatable read at all. Lua's C function `lua_getmetatable` ignores `__metatable`, so offering it would hand plugins the hidden data table behind each `var` proxy and the `_G` guard (`crates/runvil-internal/lua/src/globals.rs`, lines 1-24). `is_list` answers the one metatable question Plugins need: whether a table has the list marker.

`set_metatable` follows Lua's `setmetatable`, including its refusal to replace a protected metatable, so plugins can build table-based objects that `json.encode` still treats as plain data. It also refuses any metatable holding `__gc`, `__mode`, or `__close`. Finalizers and weak tables act whenever the garbage collector runs, and that timing differs between runs (criterion 3). `mark_list` and the `RV_TABLE_LIST` flag apply the list marker with the rules of `json.array(t)`.

### 3.5 Iteration follows the Engine's sorted order and never runs Lua

Lua leaves hash order unspecified, so the VM replaces `pairs` and `next` with a sorted walk: the list part first, then booleans, numbers, and strings in order, then any other keys (`crates/runvil-internal/lua/src/iteration.rs`, lines 1-36). `iter_begin` and `iter_next` give plugins the same order, captured once as `pairs` captures it, and they skip a key cleared during the walk. The declaration has no unordered `next`, so a plugin cannot produce output whose order changes between runs. The one gap is the VM's own: keys that are tables, functions, or userdata have no position that holds across runs, so both walks visit them last, in Lua's order.

The walk runs no metamethod, so a sealed view such as a `var` table, `argv`, or `sys` walks as empty. Lua code sees the same emptiness from `pairs` and `#`. A plugin that must read a whole structure behind a view calls `to_json`, which reads through views the way `json.encode` does without ever giving the plugin the hidden data table. That matters because `var`'s top-level data table has no guard of its own, even though every nested table in it sits behind its own proxy (`sys.rs`, lines 203-204).

### 3.6 Functions hold native data, and calls back into Lua follow Lua's limits

`new_function` pairs a callback with a `data` pointer and a `drop` function. The Engine wraps them in an mlua function, builds a fresh context and arena for each call, and calls `drop` once Lua collects the function. Functions cannot capture Lua values. An mlua closure that holds Lua values keeps them as mlua references, which the collector treats as roots, so a cycle through them would stay alive until the VM closes. A function that needs Lua state belongs on a userdata with user values and a `call` callback, which the collector traces fully.

`call` lets a plugin run a Lua function, for plugin functions that take callbacks, such as a substitution function or a tree walker. The result count is fixed, and missing results are filled with nil. A yield inside the callback fails as `RV_ERR_LUA`, because a native frame cannot be suspended, so a callback cannot reach `tools.call`, `models.infer`, or any other suspending Engine call. Lua's own nesting limit (`LUAI_MAXCCALLS`, 200 in Lua 5.5.0 `ldo.h`, line 63) bounds recursion between plugin and Lua code, and exceeding it is an ordinary `RV_ERR_LUA`. `RV_MAX_VALUES` caps arguments and results at 255, so a bad count cannot drive a huge allocation.

### 3.7 Userdata belongs to a declared type that the Engine checks

Every userdata type is a static `RvUserdataType` listed in the Plugin's `RvPluginInfo`. The Harness validates the types when the package loads, so a malformed type makes the Plugin unavailable instead of failing mid-run. The Engine builds one metatable per type and VM and never shows it to the plugin. Its `__metatable` is the type name, its `__gc` calls `drop`, and its `__tostring` defaults to the type name, because Lua's default `tostring` prints a memory address (criterion 3).

The Engine checks the type before every method and metamethod callback and raises otherwise, so a callback's `data` always belongs to its type: `u.host(5)` raises instead of reading an integer as a URL. `userdata_data` checks the same way, by the type's address, so one Plugin can never read another Plugin's data. User values hold up to four Lua values inside an instance, and the collector traces them.

The callback set is fixed: methods, `index`, `newindex`, `tostring`, `eq`, `lt`, `le`, `len`, and `call`. That covers objects with fields and methods, comparable values such as versions, and callable objects. Arithmetic, `__concat`, `__pairs`, `__close`, and `__mode` are absent, and section 4 gives the reason for each.

### 3.8 JSON conversion reuses the Engine's own rules and messages

`from_json` parses with the same code as `json.decode`: lists get the marker, null becomes `json.null`, and the 8,388,608-byte and 127-level limits apply (`crates/runvil-internal/lua/src/json/decode.rs`, line 15; `json/encode.rs`, line 22). `to_json` encodes with `json.encode`'s strict rules and reads through sealed views. In both, the plugin's own name replaces the `json.*` prefix, so an error reads `url.build: value.query.a must be JSON data, got function`.

These two calls do three jobs. They move large data in one call instead of thousands of handle operations. They give plugins a safe way to read whole structures behind views. And they keep one definition of the Lua-to-JSON rules, in the Engine.

### 3.9 Native preludes install through the prelude environment, declared under `plugins:`

A Plugin with a native prelude is an ordinary Plugin, so a prompt declares it under `plugins:` and gets the guarantees declared Plugins already have. It is required: a missing or broken Plugin refuses the run at prepare with `RequirementsUnmet` (`runvil-docs`, `src/language/04-how-a-prompt-runs.md`, "Declared Plugins are required Plugins"). Preludes install in declaration order, after the Engine globals and before any block runs, in every section VM, so walked sections, fanout arms, spawned tasks, and prompt sections loaded with `require` all see them (`src/language/05-lua-environment.md`, "Which Plugins add globals").

`open` receives the Plugin's prelude environment and sets globals on it. The Engine already treats that table's own entries as the prelude's globals: it checks each for a collision, seals each table at its top level, and copies them into `_G` (`crates/runvil-internal/lua/src/prelude.rs`, lines 16-20). Native and Lua preludes therefore share one install path. A Plugin installed as `user-input` can still define `input`, even though `user-input` is not a valid Lua name. The Plugin's optional Lua prelude runs after `open`, in the same environment, so it can wrap native functions in friendlier Lua. A failed `open` fails the run before any effect, with a message in the existing form: ``Plugin `{plugin}`: its native prelude failed: {message}``.

### 3.10 Plugin code must be pure, bounded, and unable to unwind

Some rules cannot be checked by the API, so the declaration states them as a contract, the way `Plugin::call` states its invariants (`crates/runvil-plugin/src/plugin.rs`). A plugin function's result depends only on its arguments and the package's build, because replay runs native code again. It reads no clock, no randomness, no file, and no network. The Harness records the package's name and version, which `describe_package` supplies, and the dynamic library's hash in the run's Plugin roster, so a replay against a different build is caught. A package has one version for all its Plugins, because one build produces them all.

A plugin also bounds its own work. Lua's instruction hook cannot interrupt native code, so long loops call `check_cancel` (the hook that cancels Lua code is `budget_hook` in `crates/runvil-internal/lua/src/hardening.rs`, lines 159-182). Memory a plugin allocates on its own heap falls outside the Lua memory limit, so a plugin caps its inputs, as `json.decode` caps its input at 8,388,608 bytes. `plugin_data` and the userdata types are shared by every VM on every thread, so what they point to must be thread-safe. A function's `data` belongs to one VM, which may move between threads but never runs on two at once. `drop` runs with no context and may not call `RvLua`.

## 4. The declaration leaves out every facility that can break an Engine rule

Table 1 lists the Lua facilities the declaration leaves out. Each one either fails a criterion in section 1 or is unnecessary for the target Plugins, and most have a safer replacement inside the declaration. Two exclusions have a revisit condition. Arithmetic metamethods return if a numeric Plugin is wanted (section 6.4). Host configuration returns if replay starts recording it.

Table 1. Lua facilities left out of API 1.0, why each is out, and what replaces it

| Left out | Why | Use instead |
|---|---|---|
| `lua_State*` and the stack API | Most C-API misuse is stack misuse, and the stack ties plugins to Lua's version | Values and handles |
| Registry, `luaL_ref`, persistent handles | The registry holds mlua's and the Engine's private state; persistent handles leak or dangle | Function `data`, user values |
| Light userdata other than `json.null` | mlua keys its own registry entries by light-userdata addresses | Userdata types |
| Raw set | The one write that skips `var` validation and every read-only refusal | `set`, `set_field` |
| Reading a protected metatable | Exposes the hidden data behind `var`, `argv`, `sys`, and the `_G` guard | `is_list`, `get`, `to_json` |
| The globals table | Lua prelude functions cannot see other Plugins' globals either, and ambient state hides a function's inputs | `env` in `open`; arguments |
| Code loading | The sandbox has none, and replay must re-run only known code | The Plugin's Lua prelude |
| Coroutines and yields | A yield is how the Engine asks for an effect | Call tools from the Lua prelude |
| Hooks and the debug API | The instruction hook is what makes a run cancellable; the debug library is never loaded | `check_cancel` |
| GC control, allocator, warn, panic, close, new state | mlua's allocator enforces the memory limit | Nothing |
| Unordered `next` | Hash order differs between VMs | `iter_begin`, `iter_next` |
| Pointers and address-printing `tostring` | Addresses differ between runs | `raw_equal`; type-name `tostring` |
| Implicit coercion between numbers and strings | Hidden conversions hide type errors | Read `kind`; convert in the plugin |
| `lua_arith`, `lua_compare`, `lua_concat` | They run metamethods, and Plugins compute natively | Native code |
| `__gc`, `__mode`, `__close` on tables | Finalizers and weak tables act at collection times that differ between runs | `drop` on userdata |
| Arithmetic, `__concat`, `__pairs` on userdata | No target Plugin needs them | Methods; revisit for numeric Plugins |
| Catching memory-limit errors | A plugin could turn an out-of-memory error into a wrong answer | Return the status |
| Inspecting error values | Raising unchanged needs only the status, and messages need only text | `error_message` |
| Lua values captured by functions | A closure holding Lua values hides a cycle from the collector | Userdata with user values and `call` |
| Host configuration | Replay re-runs native code but does not record configuration | One Plugin per variant |
| Logging | `log` is not visible to preludes, and a native function reports through results and errors | Results and errors |
| Tools, futures, services, file access | They belong to the tool-Plugin boundary | A later report (section 6.5) |

## 5. Three alternatives fail at least one criterion

### 5.1 A raw Lua C API table fails three criteria

The owner's first proposal passed the Lua C API itself as a struct of function pointers. It fixes the problem of two Lua copies, because the plugin links no Lua and calls the Engine's copy. It fails criterion 1: every allocating call, and every table access that can run a metamethod, can raise through `longjmp`, which skips C++ destructors and Rust `Drop` in plugin frames.

It fails criterion 2 unless the Engine filters the table, and a filter is a deny list. `lua_rawset`, the registry, `lua_sethook`, `lua_setallocf`, and `lua_yieldk` all have to be found and removed, and any one missed breaks a guard. It also fails criterion 5, because Lua's C API changes in minor releases. Lua 5.4 replaced `lua_newuserdata` with `lua_newuserdatauv` and gave `lua_resume` a result-count out-parameter (lua-src 550.1.1: `lua-5.3.6/lua.h`, lines 249 and 290; `lua-5.4.8/lua.h`, lines 270 and 311). Lua 5.5 added a seed parameter to `lua_newstate` and dropped `lua_resetthread` (`lua-5.4.8/lua.h`, lines 163-167; `lua-5.5.0/lua.h`, lines 163-166). Plugins would also have to compile against the Engine's exact `luaconf.h`, because `LUA_REGISTRYINDEX` and the number types come from it.

### 5.2 A plugin's own mlua becomes a second mlua inside the same VM

Patching `mlua-sys` so that a plugin's mlua calls through the function table would give Rust plugins the real mlua. It builds, but mlua finds its per-VM state by the address of a static variable in its own code: `EXTRA_REGISTRY_KEY` (mlua 0.12.0 `src/state/extra.rs`, line 29) and `ARRAY_METATABLE_REGISTRY_KEY` (`src/serde/mod.rs`, line 242). A copy compiled into a DLL has different addresses, so it sets up a second instance with its own list marker and its own error types.

The consequences fail criteria 2 and 3. Lists the plugin marks would not be lists to `var`, `tools.call`, or `json`, and empty lists would read back as `{}`, the exact bug the `json` work fixed. The Engine's mlua would not recognize the plugin's wrapped errors either, so `pcall` would lose their kind.

### 5.3 Building these features into the Engine works but defeats the isolation goal

Doing nothing is how `json` ships today: about 670 lines across `json.rs`, `json/encode.rs`, `json/decode.rs`, and `json/keys.rs` in `crates/runvil-internal/lua/src/`, with its tests in the Engine. It meets every safety criterion, because the code is Engine code. It fails the reason for this work. Every new feature of this kind adds crates and dependencies to the Engine's workspace, its `Cargo.lock`, its `cargo deny` audit, and its minimum-version build, and every Engine agent has to work around them.

## 6. One owner ruling and four limits remain

### 6.1 The owner must rule on native preludes against the pure-Lua prelude decision

The 2026-10-04 design record states an owner decision in section 5.6: "A prelude is just pure Lua. So the prelude can't talk to the Rust code that the capability provides." The reason it gives is that a prelude reaches its Plugin only through `tools.call`, so anything a prelude does, the prompt's own Lua or the model can do with the same tools. A native `open` function breaks the first sentence by design, because its functions are Rust code that Lua calls directly.

The recommendation is to allow native preludes only in Plugins that offer no tools, and to keep the preludes of tool Plugins pure Lua. The recorded reason still holds for tool Plugins. A native prelude's functions touch no capability: they compute from their arguments, as the Engine's own `json` global does, so they open no route around a tool. Confidence in this recommendation is medium, because the record gives only one reason for the rule, and the owner may hold others.

### 6.2 Per-call cost is unmeasured

Every operation pays an indirect call, a context check, an arena lookup in a Lua table, and an mlua protected call. That is likely to make each operation noticeably slower than the same work written directly against mlua. Confidence in that estimate is low, because nothing has been built or measured. The cost matters only for chatty table traffic. Plugins that parse text natively and return one result, or that move bulk data through `from_json`, make few calls. The spike in section 7 measures it.

### 6.3 Plugin heap memory falls outside the Lua memory limit

Each section VM's Lua heap is capped at 64 MiB by default (`DEFAULT_LUA_MEMORY_BYTES`, `crates/runvil-internal/lua/src/vm.rs`, line 48), and the cap counts only Lua allocations. A plugin that builds a large native structure from a small input can use far more memory without the Engine noticing. The declaration cannot prevent that, so the contract in section 3.10 requires plugins to cap their own inputs.

### 6.4 Numeric Plugins would need arithmetic metamethods

Arithmetic metamethods are out because no target Plugin needs them, but a decimal or big-integer Plugin would. Adding them later is an append-only change: new optional fields at the end of `RvUserdataType`, read only when the type's `size` covers them.

### 6.5 Tool Plugins need a separate boundary

This declaration covers native preludes only. Tool Plugins need futures, the per-run services, and file access across a C boundary, each as its own table. The future is the easy part. The services are harder, because `HostServices` matches providers by Rust `TypeId`, which means nothing between separately built binaries (`crates/runvil-plugin/src/service.rs`). That design belongs in its own report.

## 7. Next steps: rule, spike, guard, then build the SDK

1. **Owner (Vinnie Falco):** rule on native preludes (section 6.1), and accept or amend Table 1.
2. **Engine implementer:** build a spike of the error, arena, string, table, and function calls, the `open` hook, and `from_json`, with a test DLL that defines `url.host` (Appendix A). Measure the per-call cost against the same function written directly in mlua.
3. **Engine implementer:** commit `runvil_lua.h` with a `build-xtask` check that fails when the declaration changes without a version bump, the way `crates/runvil/public-api.txt` guards the facade.
4. **Plugin workspace:** write the Rust SDK that rebuilds mlua's familiar types (`Lua`, `Table`, `Value`, `Function`) over `RvLua`, then build the first real Plugin with a native prelude.
5. **Next report:** the tool-Plugin boundary, covering futures, services, and file access.

## References

Engine code, in the `runvil` repository:

- `crates/runvil-internal/lua/src/hardening.rs`, lines 1-2 and 159-182: the instruction hook that cancels Lua code
- `crates/runvil-internal/lua/src/__impl_coro.lua`, lines 70-108: the normalizing `pcall` and `xpcall` and their cancellation rule
- `crates/runvil-internal/lua/src/globals.rs`, lines 1-32: the `_G` guard and `RESERVED_NAMES`
- `crates/runvil-internal/lua/src/iteration.rs`, lines 1-36: the sorted `pairs` and `next`
- `crates/runvil-internal/lua/src/sys.rs`, lines 64-101 and 197-205: `var`'s guarded proxies
- `crates/runvil-internal/lua/src/prelude.rs`, lines 1-20: prelude environments, collision checks, and sealing
- `crates/runvil-internal/lua/src/json.rs` and `json/`: the `json` global and its limits
- `crates/runvil-internal/lua/src/vm.rs`, lines 48 and 249-259: the section VM's libraries and memory limit
- `crates/runvil-plugin/src/plugin.rs` and `src/service.rs`: the `Package` label (which the rename plan renames `PluginInfo`), the `Plugin` trait, and `HostServices`
- `crates/gateway/stt/whisper-ffi/src/library.rs`: the workspace's existing pattern for loading a DLL with `libloading`

Dependencies:

- mlua 0.12.0: `src/state/extra.rs`, lines 29 and 58; `src/serde/mod.rs`, line 242
- Lua 5.5.0 from lua-src 550.1.1: `ldo.c`, lines 186-196; `ldo.h`, line 63; `lua.h`, lines 163-166, 271, and 312; and `lua-5.3.6/lua.h` and `lua-5.4.8/lua.h` for the API changes in section 5.1

Language guide, in the `runvil-docs` repository:

- `src/language/04-how-a-prompt-runs.md`, "Declared Plugins are required Plugins"
- `src/language/05-lua-environment.md`, "Which Plugins add globals" and "When a prelude fails"
- `src/language/08-jump-and-call.md`, "Catching a jump with `pcall`"
- `src/language/12-tools.md`, "Declaring Plugins under `plugins:`"

Design records:

- "Runvil Plugin and Composition Design: Conclusions of the 2026-10-04 Exploration", section 5.6
- The 2026-10-18 design conversation between the owner and a Cursor agent, which produced this report

Precedents: JNI's local references and local frames, and HPy, the handle-based C API for Python extensions.

## Appendix A. A complete one-function package

This package, `acme-url`, names itself and registers one Plugin, `acme/url`. The Plugin's native prelude defines a global table `url` holding `url.host`. `find_host` stands for the Plugin's own parser.

```c
#include <stdio.h>
#include "runvil_lua.h"

static const RvLua *rv;

/* url.host(text): the host of an absolute URL, or nil. */
static RvStatus RV_CALL url_host(RvCx cx, void *data, const RvValue *args,
                                 size_t nargs) {
    (void)data;
    RvKind kind = nargs < 1 ? RV_NIL : args[0].kind;
    if (kind != RV_STRING) {
        char message[64];
        int n = snprintf(message, sizeof message,
                         "url.host expects a string, got %s",
                         rv_kind_name(kind));
        return rv->raise(cx, message, (size_t)n);
    }
    const char *text;
    size_t len;
    RvStatus status = rv->string_bytes(cx, args[0], &text, &len);
    if (status != RV_OK) {
        return status;
    }
    const char *host;
    size_t host_len;
    RvValue result = rv_nil();
    if (find_host(text, len, &host, &host_len)) {
        status = rv->new_string(cx, host, host_len, &result);
        if (status != RV_OK) {
            return status;
        }
    }
    return rv->return_values(cx, &result, 1);
}

static const RvFunctionInfo URL_HOST = {
    sizeof(RvFunctionInfo), 0u, RV_TEXT("url.host"), url_host, NULL,
};

static RvStatus RV_CALL url_open(RvCx cx, void *plugin_data, RvValue env,
                                 const char *plugin_id,
                                 size_t plugin_id_len) {
    (void)plugin_data;
    (void)plugin_id;
    (void)plugin_id_len;
    RvValue url;
    RvValue host;
    RvStatus status = rv->new_table(cx, 0u, 1u, 0u, &url);
    if (status == RV_OK) {
        status = rv->new_function(cx, &URL_HOST, NULL, &host);
    }
    if (status == RV_OK) {
        status = rv->set_field(cx, url, RV_TEXT("host"), host);
    }
    if (status == RV_OK) {
        status = rv->set_field(cx, env, RV_TEXT("url"), url);
    }
    return status;
}

static const RvPackageInfo ACME_URL_PACKAGE = {
    .size = sizeof(RvPackageInfo),
    .name = "acme-url",
    .name_len = 8u,
    .version = "0.1.0",
    .version_len = 5u,
};

static const RvPluginInfo URL_PLUGIN = {
    .size = sizeof(RvPluginInfo),
    .fqn = "acme/url",
    .fqn_len = 8u,
    .description = "Parse URLs",
    .description_len = 10u,
    .open = url_open,
};

RV_EXPORT RvStatus RV_CALL runvil_package_init(const RvHost *host,
                                               RvPackage package) {
    if (host->api_major != RV_API_MAJOR || host->api_minor < RV_API_MINOR) {
        return host->refuse(
            package, RV_TEXT("acme-url needs runvil API 1.0 or a later 1.x"));
    }
    rv = host->lua;
    RvStatus status = host->describe_package(package, &ACME_URL_PACKAGE);
    if (status != RV_OK) {
        return status;
    }
    return host->register_plugin(package, &URL_PLUGIN);
}
```

A prompt on a Host that loaded the package and installed `acme/url` under its default Plugin id, `url`, would declare the Plugin and call it like any global:

````markdown
---
name: link-host
description: Prints the host of a URL.
runvil: 0
plugins: [url]
---

# Link host

```lua
return url.host('https://www.lua.org/manual/5.5/')
```
````

Under this proposal, the run result would be:

````text
www.lua.org
````

The Engine seals `url` at its top level, as it seals every prelude table, so `url.extra = 1` raises the existing read-only refusal, naming the Plugin `url`.

---

*2026-10-18 13:55 - Claude Opus 5.5 (Cursor agent)*\
*2026-10-18 16:00 - Claude Opus 5.5 (Cursor agent), revised to the vocabulary of the Plugin registry rename*
