Secure RTP (SRTP) and UST Reference Implementations
David A. McGrew
Cisco Systems, Inc.
mcgrew@cisco.com


This package provides an implementation of the Secure Real-time
Transport Protocol (SRTP), the Universal Security Transform (UST), and
a supporting cryptographic kernel.  These mechanisms are documented in
the Internet Drafts in the doc/ subdirectory.  The SRTP API is
documented in include/srtp.h, and the library is in libsrtp.a (after
compilation).

Some notable modifications were done on this package, listed below:

* The package is now using cmake to build almost exclusively
** test are not being build yet
** docs are still being build with a makefile
** include path of "config.h" has been changed in all source and header files; it is now "srtp/config.h"

* The include files have been moved and the source files changed to reflect this change. This is to accomodate local project inclusion (using sub_directory(vendor/libsrtp) in cmake; see example below)

* installation in the system is currently not possible; I've only tested the static library output so far

* the config.h is autogenerated in the CMAKE_CURRENT_BINARY_DIR/include/srtp/

This is because it is meant to be used locally to your project and not installed on the system; I might change it to make it possible to do a cmake install as well, but not now.

Example use:

assumptions
* your project dependencies in a directory called `vendor`
* cmake variable `${libraries}` is being used in `target_link_libraries()`
* cmake variable `${include_dirs}` is being using in `set_target_properties(myproj INCLUDE_DIRECTORIES ${include_dirs})`

`add_subdirectory(vendor/libsrtp)`
`list(APPEND include_dirs "${CMAKE_CURRENT_SOURCE_DIR}/vendor/libsrtp/include")`

This is for the autogenerated config file (config.h)
`list(APPEND include_dirs "${CMAKE_BINARY_DIR}/vendor/libsrtp/include")`

`list(APPEND libraries "libsrtp")`

use in your code like so:
```C++
#include <srtp/srtp.h>

int main(int argc, char** argv)
{
	srtp_t session;
	srtp_policy_t policy;
	uint8_t key[30];

	crypto_policy_set_rtp_default(&policy.rtp);
	crypto_policy_set_rtcp_default(&policy.rtcp);

	crypto_get_random(key, 30);

	srtp_init();

	...
}
```

don't forget to read the original README as well