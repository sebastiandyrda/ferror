# ferror
A library to assist with error handling in Fortran projects.

## Status
![Build Status](https://travis-ci.org/jchristopherson/ferror.svg?branch=master)

## Usage

```fortran
program example
    use ferror
    use, intrinsic :: iso_fortran_env, only : int32
    implicit none

    ! Variables
    type(errors) :: err_mgr

    ! Ensure the error reporting doesn't terminate the application.  The default
    ! behavior terminates the application.
    call err_mgr%set_exit_on_error(.false.)

    ! Don't print the error message to the command line.  The default behavior
    ! prints the error information to the command line.
    call err_mgr%set_suppress_printing(.true.)

    ! Call the routine that causes the error
    call causes_error(err_mgr)

    ! Print the error information
    print '(A)', "An error occurred in the following subroutine: " // &
        err_mgr%get_error_fcn_name()
    print '(A)', "The error message is: " // err_mgr%get_error_message()
    print '(AI0)', "The error code is: ", err_mgr%get_error_flag()
contains

! The purpose of this subroutine is to simply trigger an error condition.
subroutine causes_error(err)
    ! Arguments
    class(errors), intent(inout) :: err

    ! Define an error flag
    integer(int32), parameter :: error_flag = 200

    ! Trigger the error condition
    call err%report_error(&
        "causes_error", &                   ! The subroutine or function name
        "This is a test error message.", &  ! The error message.
        error_flag)                         ! The error flag
end subroutine

end program
```
The above program produces the following output.
```text
An error occurred in the following subroutine: causes_error
The error message is: This is a test error message.
The error code is: 200
```

## C Usage
```c
#include <stdio.h>
#include "ferror.h"

void causes_error(errorhandler *err);


int main(void) {
    // Variables
    errorhandler err_mgr;
    char fname[256], msg[256];
    int flag, fnamelength = 256, msglength = 256;

    // Initialization
    alloc_errorhandler(&err_mgr);

    // Ensure the error reporting doesn't terminate the application
    set_exit_on_error(&err_mgr, false);

    // Don't print the error message to the command line
    set_suppress_printing(&err_mgr, true);

    // Call the routine that causes the error
    causes_error(&err_mgr);

    // Retrieve the error information
    get_error_fcn_name(&err_mgr, fname, &fnamelength);
    get_error_message(&err_mgr, msg, &msglength);
    flag = get_error_flag(&err_mgr);

    // Print the error information
    printf("An error occurred in the following subroutine: %s\nThe error message is: %s\nThe error code is: %i\n",
        fname, msg, flag);

    // End
    free_errorhandler(&err_mgr);
    return 0;
}

void causes_error(errorhandler *err) {
    report_error(err,                       // The errorhandler object
        "causes_error",                     // The function name
        "This is a test error message.",    // The error message
        200);                               // The error flag
}
```
The above program produces the following output.
```text
An error occurred in the following subroutine: causes_error
The error message is: This is a test error message.
The error code is: 200
```

## Documentation
Documentation can be found [here](http://htmlpreview.github.io/?https://github.com/jchristopherson/ferror/blob/master/doc/html/index.html).

## Build Instructions
This library utilizes [CMake](https://cmake.org/) to facilitate its build.  Using CMake is as simple as issuing the following commands.
- cmake ...
- make
- make install

See [Running CMake](https://cmake.org/runningcmake/) for more details on the use of CMake.
