CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE "RELEASE")
ENDIF()

if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  list(APPEND CMAKE_PREFIX_PATH /opt/local/opt/qt5)
endif()


if(CMAKE_CXX_FLAGS_RELEASE MATCHES DNDEBUG)
  string(REPLACE "-DNDEBUG" "" CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")
  string(REPLACE "-DNDEBUG" "" CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE}")
endif()

if(CMAKE_CXX_FLAGS_RELWITHDEBINFO MATCHES DNDEBUG)
  string(REPLACE "-DNDEBUG" "" CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELEASE}")
  string(REPLACE "-DNDEBUG" "" CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_C_FLAGS_RELEASE}")
endif()

project(mazacoin-qt)

set(OPENSSL_VER "1.0.2n")
set(BDB_VER "4.8.30")

#---------------------------------------------------------------------------------------------------------
# Code coverage
#---------------------------------------------------------------------------------------------------------
if (${CMAKE_CXX_COMPILER_ID} STREQUAL "Clang")
    set(CMAKE_XCODE_ATTRIBUTE_GCC_VERSION "com.apple.compilers.llvm.clang.1_0")
    # Setup Code Coverage for XCode when requested
    if ($ENV{COVERAGE})
        message(STATUS "********* Setting up Xcode compile for Code coverage **********")
        set(CMAKE_XCODE_ATTRIBUTE_GCC_INSTRUMENT_PROGRAM_FLOW_ARCS "YES")
        set(CMAKE_XCODE_ATTRIBUTE_GCC_GENERATE_TEST_COVERAGE_FILES "YES")
        # Works for Clang->
				#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -coverage")
    endif()
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 ")


IF (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	add_compile_options(-Wno-potentially-evaluated-expression -Wno-unused-const-variable -Wno-unused-function -Wno-deprecated-declarations -Wno-inaccessible-base -Wno-unused-local-typedefs -Wno-logical-op-parentheses -Wno-shift-negative-value -Wno-array-bounds)
	add_definitions("-DOS_MACOSX -DLEVELDB_PLATFORM_POSIX")
	add_definitions("-DENABLE_WALLET")
#	add_definitions("-Werror")
else()
	add_definitions("-Wno-cpp -Wno-unused-function")
	set(ANL anl)
endif()

# Disable -DHAVE_BUILD_INFO for now to avoid compiler error
add_definitions("-DHAVE_CONFIG_H -DBOOST_SPIRIT_THREADSAFE -D__STDC_FORMAT_MACROS -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2")

add_definitions("-Wall -Wextra -Wformat -Wformat-security -Wno-unused-parameter -Wstack-protector -fstack-protector-all -fPIC -fvisibility=hidden")

# run autogen.sh if missing header files from configure on Linux/Mac
if (EXISTS	"${CMAKE_CURRENT_SOURCE_DIR}/configure")
else()	
  execute_process(
	COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/autogen.sh
	WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
	)
endif()

# run configure if pivx_config.h doesn't exist, For Linux add "--with-incompatible-bdb"
if (EXISTS	"${CMAKE_CURRENT_SOURCE_DIR}/src/config/maza-config.h")
else()	
	IF (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	  execute_process(
		COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/configure 
		WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
		)
else()
	  execute_process(
		COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/configure  --with-incompatible-bdb
		WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
		)
	endif()
endif()


SET(COMMON_A
	./src/protocol.cpp
	./src/pubkey.cpp
	./src/netbase.cpp
	./src/keystore.cpp
	./src/key.cpp
	./src/hash.cpp
	./src/core_write.cpp
	./src/core_read.cpp
	./src/compressor.cpp
	./src/coins.cpp
	./src/chainparams.cpp
	./src/base58.cpp
	./src/warnings.cpp
	)

SET(UTIL_A
	./src/utiltime.cpp
	./src/utilmoneystr.cpp
	./src/utilstrencodings.cpp
	./src/util.cpp
	./src/sync.cpp
	./src/uint256.cpp
	./src/arith_uint256.cpp
	./src/rpc/protocol.cpp
	./src/random.cpp
	./src/clientversion.cpp
	./src/chainparamsbase.cpp
	./src/support/lockedpool.cpp
	./src/support/cleanse.cpp
	./src/threadinterrupt.cpp
	)




SET(SERVER_A
	./src/amount.cpp
	./src/addrman.cpp
	./src/addrdb.cpp
	./src/bloom.cpp
	./src/blockencodings.cpp
	./src/chain.cpp
	./src/checkpoints.cpp
	./src/config.cpp
	./src/globals.cpp
	./src/httprpc.cpp
	./src/httpserver.cpp
	./src/init.cpp
	./src/dbwrapper.cpp
	./src/merkleblock.cpp
	./src/miner.cpp
	./src/net.cpp
	./src/net_processing.cpp
    ./src/rpc/abc.cpp
	./src/rpc/blockchain.cpp
	./src/rpc/client.cpp
	./src/rpc/mining.cpp
	./src/rpc/misc.cpp
	./src/rpc/net.cpp
	./src/rpc/protocol.cpp
	./src/rpc/rawtransaction.cpp
	./src/rpc/server.cpp
    
	./src/noui.cpp
    ./src/policy/fees.cpp
    ./src/policy/policy.cpp
	./src/pow.cpp
	./src/rest.cpp
    
	./src/timedata.cpp
	./src/torcontrol.cpp
	./src/txdb.cpp
	./src/txmempool.cpp
	./src/ui_interface.cpp
	./src/validation.cpp
	./src/validationinterface.cpp
	./src/versionbits.cpp
    
	./src/scheduler.cpp
	./src/dbwrapper.cpp
	./src/netaddress.cpp
	./src/protocol.cpp
#	./src/rpc/net.cpp
#	./src/rpcclient.cpp
	./src/wallet/rpcdump.cpp
	./src/wallet/rpcwallet.cpp
	)


SET(CONSENSUS
	./src/utilstrencodings.cpp
	./src/pubkey.cpp
	./src/uint256.cpp
	./src/arith_uint256.cpp
	./src/hash.cpp
	./src/consensus/merkle.cpp
)	

SET(CONSENSUS_CRYPTO
	./src/crypto/aes.cpp
	./src/crypto/chacha20.cpp
	./src/crypto/hmac_sha512.cpp
	./src/crypto/hmac_sha256.cpp
	./src/crypto/ripemd160.cpp
	./src/crypto/sha256.cpp
	./src/crypto/sha1.cpp
	./src/crypto/sha512.cpp
)

SET(SOURCES
	./src/univalue/lib/univalue_write.cpp
	./src/compat/strnlen.cpp
	./src/compat/glibc_sanity.cpp
	./src/compat/glibcxx_sanity.cpp
	./src/miner.cpp
	./src/primitives/block.cpp
	./src/primitives/transaction.cpp
	./src/script/bitcoinconsensus.cpp
	./src/script/interpreter.cpp
	./src/script/script.cpp
	./src/script/script_error.cpp
	./src/script/sigcache.cpp
	./src/script/sign.cpp
	./src/script/standard.cpp
	./src/script/ismine.cpp
	./src/txdb.cpp
	./src/txmempool.cpp
  ./src/validation.cpp
  ./src/validationinterface.cpp
  ./src/versionbits.cpp
	./src/univalue/lib/univalue.cpp
	./src/univalue/lib/univalue_read.cpp
	./src/wallet/wallet.cpp
	./src/wallet/finaltx.cpp
	./src/wallet/crypter.cpp
	./src/wallet/db.cpp
	./src/wallet/walletdb.cpp
	)

SET(ZMQ_SOURCES
  ./src/zmq/zmqnotificationinterface.cpp
  ./src/zmq/zmqpublishnotifier.cpp
  ./src/zmq/zmqabstractnotifier.cpp
  )


IF (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  set(OPENSSL_ROOT_DIR "/usr/local/Cellar/openssl/${OPENSSL_VER}/")
  set(BDB_ROOT_DIR "/usr/local/Cellar/berkeley-db4/${BDB_VER}/")
endif()

find_package(OpenSSL)
if (OPENSSL_FOUND)
  message(STATUS "Found OpenSSL")
endif()  

find_package( Boost COMPONENTS system filesystem thread program_options chrono)
link_directories ( ${Boost_LIBRARY_DIRS} )

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${CMAKE_CURRENT_SOURCE_DIR}/src/config
  ${CMAKE_CURRENT_SOURCE_DIR}/src/obj
  ${Boost_INCLUDE_DIRS}
  ${OPENSSL_INCLUDE_DIR}
  ${BDB_ROOT_DIR}/include
  ${CMAKE_CURRENT_SOURCE_DIR}/src/leveldb/include
  ${CMAKE_CURRENT_SOURCE_DIR}/src/univalue/include
  ${CMAKE_CURRENT_SOURCE_DIR}/src/leveldb/helpers/memenv
  ${CMAKE_CURRENT_SOURCE_DIR}/src/secp256k1
  ${CMAKE_CURRENT_SOURCE_DIR}/src/secp256k1/include
  ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/
  ${CMAKE_CURRENT_SOURCE_DIR}/src/qt/forms
  )

#file(GLOB_RECURSE HEADERS *.h)
file(GLOB HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/src/*.h)

file(GLOB CRYPTO_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/src/crypto/*.h)
source_group("CryptoHeaders" FILES ${CRYPTO_HEADERS})

file(GLOB PRIMITIVE_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/src/primitives/*.h)
source_group("PrimHeaders" FILES ${PRIMITIVE_HEADERS})

add_library(coin ${HEADERS} ${CRYPTO_HEADERS} ${PRIMITIVE_HEADERS} ${SOURCES} ${ZMQ_SOURCES} ${COMMON_A} ${UTIL_A} ${SERVER_A} ${CONSENSUS} ${CONSENSUS_CRYPTO})
set_property(TARGET coin PROPERTY CXX_STANDARD 11)

add_subdirectory(src)

set(SECP256 ./src/secp256k1/src/secp256k1.c)
add_library(secp256k1 ${SECP256})

link_directories(${BDB_ROOT_DIR}/lib)

add_executable(maza-cli ${CMAKE_CURRENT_SOURCE_DIR}/src/maza-cli.cpp)
add_executable(maza-tx ${CMAKE_CURRENT_SOURCE_DIR}/src/maza-tx.cpp)
add_executable(mazad ${CMAKE_CURRENT_SOURCE_DIR}/src/mazad.cpp)

target_link_libraries(maza-cli coin db_cxx db secp256k1 gmp leveldb ${OPENSSL_LIBRARIES} ${Boost_LIBRARIES} ${ANL} miniupnpc event event_pthreads pthread )
target_link_libraries(maza-tx coin secp256k1 coin leveldb ${OPENSSL_LIBRARIES} ${Boost_LIBRARIES} ${ANL} pthread)

IF (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	target_link_libraries(mazad coin db_cxx db secp256k1 gmp leveldb ${OPENSSL_LIBRARIES} ${Boost_LIBRARIES} ${ANL} zmq miniupnpc event event_pthreads pthread )
else()
	target_link_libraries(mazad coin db_cxx db secp256k1 leveldb ${OPENSSL_LIBRARIES} ${Boost_LIBRARIES} ${ANL} pthread)
endif()

add_subdirectory(src/qt)

#---------------------------------------------------------------------------------------------------------
# Create a target in Xcode to setup coverage script
#---------------------------------------------------------------------------------------------------------
if ($ENV{COVERAGE})
    add_custom_target(xcov COMMAND ./XcodeCoverage/exportenv.sh )
endif()

#add_subdirectory(src/test)
  


