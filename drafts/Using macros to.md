Using macros to

* minimize potential future refactoring effort

* implement generics

* implement closures:

* test pre-/post- conditions

* be clever:

  * ```c
    #define SPRINTF_D(_buffer_, _i_) sprintf(_buffer_, "%d", _i_)
    #define SPRINTF_U(_buffer_, _u_) sprintf(_buffer_, "%u", _u_)
    ```

* define functions:

  * ```c
    #define VERIFY(_x_) if (!(_x_)) {                                                       \
            notify_assertion_violation(__FILE__, __LINE__, "Failed to verify: " #_x_ "\n"); \
            exit(ERR_UNREACHABLE);                                                          \
        }                                                           
    
    #define ENSURE(_x_) VERIFY(_x_)
    ```

  * 



