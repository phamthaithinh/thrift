#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements. See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership. The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License. You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied. See the License for the
# specific language governing permissions and limitations
# under the License.
#


# Find required packages
set(Boost_USE_STATIC_LIBS ON) # Force the use of static boost test framework
find_package(Boost 1.53.0 REQUIRED COMPONENTS unit_test_framework)
include_directories("${Boost_INCLUDE_DIR}")

#Make sure gen-cpp files can be included
include_directories("${CMAKE_CURRENT_BINARY_DIR}")

# Create the thrift C++ test library
set(testgencpp_SOURCES
    gen-cpp/DebugProtoTest_types.cpp
    gen-cpp/DebugProtoTest_types.h
    gen-cpp/EnumTest_types.cpp
    gen-cpp/EnumTest_types.h
    gen-cpp/OptionalRequiredTest_types.cpp
    gen-cpp/OptionalRequiredTest_types.h
    gen-cpp/Recursive_types.cpp
    gen-cpp/Recursive_types.h
    gen-cpp/ThriftTest_types.cpp
    gen-cpp/ThriftTest_types.h
    gen-cpp/TypedefTest_types.cpp
    gen-cpp/TypedefTest_types.h
    ThriftTest_extras.cpp
    DebugProtoTest_extras.cpp
)

add_library(testgencpp STATIC ${testgencpp_SOURCES})
target_link_libraries(testgencpp thrift)

set(processortest_SOURCES
    gen-cpp/ChildService.cpp
    gen-cpp/ChildService.h
    gen-cpp/ParentService.cpp
    gen-cpp/ParentService.h
    gen-cpp/proc_types.cpp
    gen-cpp/proc_types.h
)


add_executable(Benchmark Benchmark.cpp)
target_link_libraries(Benchmark testgencpp)
add_test(NAME Benchmark COMMAND Benchmark)

set(UnitTest_SOURCES
    UnitTestMain.cpp
    TMemoryBufferTest.cpp
    TBufferBaseTest.cpp
    Base64Test.cpp
    ToStringTest.cpp
    TypedefTest.cpp
)

if(NOT WITH_BOOSTTHREADS AND NOT WITH_STDTHREADS)
    list(APPEND UnitTest_SOURCES RWMutexStarveTest.cpp)
endif()

add_executable(UnitTests ${UnitTest_SOURCES})
target_link_libraries(UnitTests testgencpp thrift ${Boost_LIBRARIES})
add_test(NAME UnitTests COMMAND UnitTests)


if(WITH_ZLIB)
add_executable(TransportTest TransportTest.cpp)
target_link_libraries(TransportTest
    testgencpp
    thriftz
    ${Boost_LIBRARIES}
    ${ZLIB_LIBRARIES}
)
add_test(NAME TransportTest COMMAND TransportTest)

add_executable(ZlibTest ZlibTest.cpp)
target_link_libraries(ZlibTest
    testgencpp
    thriftz
    ${Boost_LIBRARIES}
    ${ZLIB_LIBRARIES}
)
add_test(NAME ZlibTest COMMAND ZlibTest)
endif(WITH_ZLIB)


add_executable(EnumTest EnumTest.cpp)
target_link_libraries(EnumTest
    testgencpp
    ${Boost_LIBRARIES}
)
add_test(NAME EnumTest COMMAND EnumTest)

add_executable(TFileTransportTest TFileTransportTest.cpp)
target_link_libraries(TFileTransportTest
    testgencpp
    ${Boost_LIBRARIES}
)
add_test(NAME TFileTransportTest COMMAND TFileTransportTest)

add_executable(TFDTransportTest TFDTransportTest.cpp)
target_link_libraries(TFDTransportTest
    thrift
)
add_test(NAME TFDTransportTest COMMAND TFDTransportTest)

add_executable(TPipedTransportTest TPipedTransportTest.cpp)
target_link_libraries(TPipedTransportTest
    thrift
)
add_test(NAME TPipedTransportTest COMMAND TPipedTransportTest)

set(AllProtocolsTest_SOURCES
    AllProtocolTests.cpp
    AllProtocolTests.tcc
    GenericHelpers
    )

add_executable(AllProtocolsTest ${AllProtocolsTest_SOURCES})
target_link_libraries(AllProtocolsTest
    testgencpp
)
add_test(NAME AllProtocolsTest COMMAND AllProtocolsTest)

add_executable(DebugProtoTest DebugProtoTest.cpp)
target_link_libraries(DebugProtoTest
    testgencpp
)
add_test(NAME DebugProtoTest COMMAND DebugProtoTest)

add_executable(JSONProtoTest JSONProtoTest.cpp)
target_link_libraries(JSONProtoTest
    testgencpp
)
add_test(NAME JSONProtoTest COMMAND JSONProtoTest)

add_executable(OptionalRequiredTest OptionalRequiredTest.cpp)
target_link_libraries(OptionalRequiredTest
    testgencpp
)
add_test(NAME OptionalRequiredTest COMMAND OptionalRequiredTest)

add_executable(RecursiveTest RecursiveTest.cpp)
target_link_libraries(RecursiveTest
    testgencpp
)
add_test(NAME RecursiveTest COMMAND RecursiveTest)

add_executable(SpecializationTest SpecializationTest.cpp)
target_link_libraries(SpecializationTest
    testgencpp
)
add_test(NAME SpecializationTest COMMAND SpecializationTest)

set(concurrency_test_SOURCES
    concurrency/Tests.cpp
    concurrency/ThreadFactoryTests.h
    concurrency/ThreadManagerTests.h
    concurrency/TimerManagerTests.h
)
add_executable(concurrency_test ${concurrency_test_SOURCES})
target_link_libraries(concurrency_test
    thrift
)
add_test(NAME concurrency_test COMMAND concurrency_test)

set(link_test_SOURCES
    link/LinkTest.cpp
    link/TemplatedService1.cpp
    link/TemplatedService2.cpp
    gen-cpp/ParentService.h
)
add_executable(link_test ${link_test_SOURCES})
add_test(NAME link_test COMMAND link_test)

if(WITH_LIBEVENT)
set(processor_test_SOURCES
    processor/ProcessorTest.cpp
    processor/EventLog.cpp
    processor/ServerThread.cpp
    processor/EventLog.h
    processor/Handlers.h
    processor/ServerThread.h
)
add_executable(processor_test ${processor_test_SOURCES})
target_link_libraries(processor_test
    thrift
    thriftnb
    ${Boost_LIBRARIES}
)
add_test(NAME processor_test COMMAND processor_test)
endif()

if(OPENSSL_FOUND AND WITH_OPENSSL)
add_executable(OpenSSLManualInitTest OpenSSLManualInitTest.cpp)
target_link_libraries(OpenSSLManualInitTest
    thrift
    ${OPENSSL_LIBRARIES}
    ${Boost_LIBRARIES}
)
add_test(NAME OpenSSLManualInitTest COMMAND OpenSSLManualInitTest)
endif()

#
# Common thrift code generation rules
#


add_custom_command(OUTPUT gen-cpp/DebugProtoTest_types.cpp gen-cpp/DebugProtoTest_types.h
    COMMAND thrift-compiler --gen cpp:dense ${PROJECT_SOURCE_DIR}/test/DebugProtoTest.thrift
)

add_custom_command(OUTPUT gen-cpp/EnumTest_types.cpp gen-cpp/EnumTest_types.h
    COMMAND thrift-compiler --gen cpp ${PROJECT_SOURCE_DIR}/test/EnumTest.thrift
)

add_custom_command(OUTPUT gen-cpp/TypedefTest_types.cpp gen-cpp/TypedefTest_types.h
    COMMAND thrift-compiler --gen cpp ${PROJECT_SOURCE_DIR}/test/TypedefTest.thrift
)

add_custom_command(OUTPUT gen-cpp/OptionalRequiredTest_types.cpp gen-cpp/OptionalRequiredTest_types.h
    COMMAND thrift-compiler --gen cpp:dense ${PROJECT_SOURCE_DIR}/test/OptionalRequiredTest.thrift
)

add_custom_command(OUTPUT gen-cpp/Recursive_types.cpp gen-cpp/Recursive_types.h
    COMMAND thrift-compiler --gen cpp ${PROJECT_SOURCE_DIR}/test/Recursive.thrift
)

add_custom_command(OUTPUT gen-cpp/Service.cpp gen-cpp/StressTest_types.cpp
    COMMAND thrift-compiler --gen cpp:dense ${PROJECT_SOURCE_DIR}/test/StressTest.thrift
)

add_custom_command(OUTPUT gen-cpp/SecondService.cpp gen-cpp/ThriftTest_constants.cpp gen-cpp/ThriftTest.cpp gen-cpp/ThriftTest_types.cpp gen-cpp/ThriftTest_types.h
    COMMAND thrift-compiler --gen cpp:dense ${PROJECT_SOURCE_DIR}/test/ThriftTest.thrift
)

add_custom_command(OUTPUT gen-cpp/ChildService.cpp gen-cpp/ChildService.h gen-cpp/ParentService.cpp gen-cpp/ParentService.h gen-cpp/proc_types.cpp gen-cpp/proc_types.h
    COMMAND thrift-compiler --gen cpp:templates,cob_style ${CMAKE_CURRENT_SOURCE_DIR}/processor/proc.thrift
)
