## Review Notes
### Example for JSON
```
{
    "Breed": "Dog",
    "Sound": "Sometimes \\"Wow\\""
}
```

### Compile Parameters
`-Wall`: enables all compiler's warning messages. This option should always be used, in order to generate better code.  
`-ansi`: equivalent to `-std=c89`, tells the compiler to implement the ANSI language option. This turns off certain "features" of GCC which are incompatible with the ANSI standard.  
`-pedantic`: used in conjunction with -ansi, this tells the compiler to be adhere strictly to the ANSI standard, rejecting any code which is not compliant.  

Demo:
```
cmake_minimum_required (VERSION 2.6)
project (json-parser C)

if (CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ansi -pedantic -Wall")
endif()

add_library(leptjson leptjson.c)
add_executable(leptjson_test test.c)
target_link_libraries(leptjson_test leptjson)
```

### C Header
```
#ifndef LEPTJSON_H__
#define LEPTJSON_H__

/* ... */

#endif /* LEPTJSON_H__ */
```

### Namespace
C language does not have the namespace keyword of C++, and generally it uses the abbreviation of the project as the prefix of the identifier. e.g. `LEPT_NULL`

### Starting Code
```
typedef enum { LEPT_NULL, LEPT_FALSE, LEPT_TRUE, LEPT_NUMBER, LEPT_STRING, LEPT_ARRAY, LEPT_OBJECT } lept_type;

typedef struct {
    lept_type type;
} lept_value;
```

In C, a struct is declared in the form 'struct X {}', and variables are defined as 'struct X x;'. To make it easier to use, the code above uses 'typedef'.

Most basic API:
```
int lept_parse(lept_value* v, const char* json);
lept_type lept_get_type(const lept_value* v);
```
json is a null-terminated string, it is modified by const char* since we should not edit it.

Usage for the above API:
```
lept_value v;
const char json[] = ...;
int ret = lept_parse(&v, json);
```

The return value should be one of the below enums:
```
enum {
    LEPT_PARSE_OK = 0,
    LEPT_PARSE_EXPECT_VALUE,
    LEPT_PARSE_INVALID_VALUE,
    LEPT_PARSE_ROOT_NOT_SINGULAR
};
```

Basic implementation for lept_parse
```
typedef struct {
    const char* json;
} lept_context;

int lept_parse(lept_value* v, const char* json) {
    lept_context c;
    assert(v != NULL);
    c.json = json;
    v->type = LEPT_NULL;
    lept_parse_whitespace(&c);
    return lept_parse_value(&c, v);
}

```
To reduce the number of parameters passed between parsing functions, we put all of this data into a 'lept_context' struct.

### JSON Grammar
Based on RFC-7159, in the form of ABNF:
```
JSON-text = ws value ws
ws = *(%x20 / %x09 / %x0A / %x0D)
value = null / false / true 
null  = "null"
false = "false"
true  = "true"
```

%xhh is char of base-16.  
%x20 = U+0020 = SPACE  
%x09 = U+0009 = TAB  
%x0A = U+000A = LF  
%x0D = U+000D = CR  

### Simple TDD Framework
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "leptjson.h"

static int main_ret = 0;
static int test_count = 0;
static int test_pass = 0;

#define EXPECT_EQ_BASE(equality, expect, actual, format) \
    do { \
        test_count++; \
        if (equality) \
            test_pass++; \
        else { \
            fprintf(stderr, "%s:%d: expect: " format " actual: " format "\n", __FILE__, __LINE__, expect, actual); \
            main_ret = 1; \
        } \
    } while(0)

#define EXPECT_EQ_INT(expect, actual) EXPECT_EQ_BASE((expect) == (actual), expect, actual, "%d")

static void test_parse_null() {
    lept_value v;
    v.type = LEPT_TRUE;
    EXPECT_EQ_INT(LEPT_PARSE_OK, lept_parse(&v, "null"));
    EXPECT_EQ_INT(LEPT_NULL, lept_get_type(&v));
}

/* ... */

static void test_parse() {
    test_parse_null();
    /* ... */
}

int main() {
    test_parse();
    printf("%d/%d (%3.2f%%) passed\n", test_pass, test_count, test_pass * 100.0 / test_count);
    return main_ret;
}
```

Details for printf:  
```
#include <stdio.h>

int main()
{
    printf("Hello World" " Have" " Fun" "\n");
    return 0;
}
```

Trick for multiline macro:  
```
// Wrong way 1
#define M() a(); b()

// Original
if (cond)
    M();
else
    c();

// Preprocessed
if (cond)
    a(); b(); /* b(); is out of if              */
else          /* <- else lacks corresponding if */
    c();
```

```
// Wrong way 2
#define M() { a(); b(); }

// Preprocessed
if (cond)
    { a(); b(); }; /* the last semicolon indicates the end of if */
else               /* <- else lacks corresponding if             */
    c();
```

```
// Correct way
#define M() do { a(); b(); } while(0)

// Preprocessed
if (cond)
    do { a(); b(); } while(0);
else
    c();
```

### C Details
In C89, there is no inline function.  
Inline function was introduced in C99.

```
static inline int foo()
{
    return 42;
}

int main()
{
    int ret;
  
    // inline function call
    ret = foo();
  
    printf("Output is: %d\n", ret);
    
    return 0;
}
```

Compiler provided macro: `__LINE__` and `__FILE__`.