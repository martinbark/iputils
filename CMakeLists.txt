cmake_minimum_required( VERSION 2.8 )
project(iputils C)

#path to local cmake modules
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/modules)
include(CheckCSourceCompiles)

add_definitions(-D_GNU_SOURCE)

SET(LIBGCRYPT_CONFIG libgcrypt-config CACHE STRING "Path to libgcrypt-config")

option(WITHOUT_IFADDRS "Do not use getifaddrs" OFF)
if(WITHOUT_IFADDRS)
    add_definitions(-DWITHOUT_IFADDRS)
endif(WITHOUT_IFADDRS)

option(RDISC_SERVER "rdisc server (-r option) support" OFF)
if(RDISC_SERVER)
    add_definitions(-DRDISC_SERVER)
endif(RDISC_SERVER)

set(ARPING_DEFAULT_DEVICE "" CACHE STRING "arping default device (e.g. eth0)")
if(ARPING_DEFAULT_DEVICE)
    add_definitions(-DDEFAULT_DEVICE=\"${ARPING_DEFAULT_DEVICE}\")
endif(ARPING_DEFAULT_DEVICE)

# libsysfs
add_definitions(-DUSE_SYSFS)

# libresolv
option(USE_RESOLV "Resolv library for ping6" ON)
if(USE_RESOLV)
    find_package(libresolv)
endif(USE_RESOLV)

# libcap
option(USE_CAP "Capability support (with libcap)" ON)
if(USE_CAP)
    find_package(libcap)
    if(CAP_FOUND)
        add_definitions(-DCAPABILITIES)
    endif(CAP_FOUND)
endif(USE_CAP)

# libidn2
option(USE_IDN "IDN2 support" ON)
if(USE_IDN)
    find_package(libidn2)
    if(IDN2_FOUND)
        add_definitions(-DUSE_IDN)
    endif(IDN2_FOUND)
endif(USE_IDN)

# libgcrypt
option(USE_GCRYPT "libgcrypt library for ipv6 ping" OFF)
if(USE_GCRYPT)
    find_package(libgcrypt)
endif(USE_GCRYPT)

# nettle
option(USE_NETTLE "nettle library for ipv6 ping" ON)
if(USE_NETTLE)
    find_package(nettle)
endif(USE_NETTLE)

if(GCRYPT_FOUND)
    MESSAGE(STATUS "Using GCRYPT")
    add_definitions(-DUSE_GCRYPT)
    SET(CRYPTO_LIBRARIES ${GCRYPT_LIBRARIES})
else(GCRYPT_FOUND)
    if(NETTLE_FOUND)
        MESSAGE(STATUS "Using NETTLE")
        add_definitions(-DUSE_NETTLE)
        SET(CRYPTO_LIBRARIES ${NETTLE_LIBRARIES})
    endif(NETTLE_FOUND)
endif(GCRYPT_FOUND)

# check if libm is needed for finite()
check_c_source_compiles("
    #include <math.h>
    int main() {return finite(0.0);}
" HAVE_FINITE)
if(NOT HAVE_FINITE)
    list(APPEND CMAKE_REQUIRED_LIBRARIES m)
    check_c_source_compiles("
        #include <math.h>
        int main() {return finite(0.0);}
    " HAVE_FINITE_LIBM)
    if(HAVE_FINITE_LIBM)
        set(M_LIBRARIES m)
    else()
        message(FATAL_ERROR "finite() not found")
    endif()
endif()

# check if librt is needed for clock_gettime()
check_c_source_compiles("
    #include <time.h>
    int main() {struct timespec now; return clock_gettime(CLOCK_MONOTONIC, &now);}
" HAVE_CLOCK_GETTIME)
if(NOT HAVE_CLOCK_GETTIME)
    list(APPEND CMAKE_REQUIRED_LIBRARIES rt)
    check_c_source_compiles("
        #include <time.h>
        int main() {struct timespec now; return clock_gettime(CLOCK_MONOTONIC, &now);}
    " HAVE_CLOCK_GETTIME_LIBRT)
    if(HAVE_CLOCK_GETTIME_LIBRT)
        set(RT_LIBRARIES rt)
    else()
        message(FATAL_ERROR "clock_gettime() not found")
    endif()
endif()

# ping
add_executable(ping ping.c ping_common.c ping6_common.c)
target_link_libraries(ping ${RESOLV_LIBRARIES} ${IDN2_LIBRARIES} ${CAP_LIBRARIES} ${CRYPTO_LIBRARIES} ${M_LIBRARIES})
install (TARGETS ping DESTINATION bin)

# tracepath
add_executable(tracepath tracepath.c)
target_link_libraries(tracepath ${IDN2_LIBRARIES})
install (TARGETS tracepath DESTINATION bin)

# traceroute6
add_executable(traceroute6 traceroute6.c)
target_link_libraries(traceroute6 ${IDN2_LIBRARIES} ${CAP_LIBRARIES})
install (TARGETS traceroute6 DESTINATION bin)

# clockdiff
add_executable(clockdiff clockdiff.c)
target_link_libraries(clockdiff ${CAP_LIBRARIES})
install (TARGETS clockdiff DESTINATION bin)

# rdisc
add_executable(rdisc rdisc.c)
install (TARGETS rdisc DESTINATION bin)

# arping
add_executable(arping arping.c)
target_link_libraries(arping ${IDN2_LIBRARIES} ${CAP_LIBRARIES} ${SYSFS_LIBRARIES} ${RT_LIBRARIES})
install (TARGETS arping DESTINATION bin)

# tftpd
add_executable(tftpd tftpd.c tftpsubs.c)
install (TARGETS tftpd DESTINATION bin)

# rarpd
add_executable(rarpd rarpd.c)
install (TARGETS rarpd DESTINATION bin)
