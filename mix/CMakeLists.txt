cmake_policy(SET CMP0015 NEW)
# let cmake autolink dependencies on windows
cmake_policy(SET CMP0020 NEW)
# this policy was introduced in cmake 3.0
# remove if, once 3.0 will be used on unix
if (${CMAKE_MAJOR_VERSION} GREATER 2)
	cmake_policy(SET CMP0043 OLD)
endif()

set(CMAKE_INCLUDE_CURRENT_DIR ON)

aux_source_directory(. SRC_LIST)

include_directories(BEFORE ${JSONCPP_INCLUDE_DIRS})
include_directories(${Boost_INCLUDE_DIRS})
include_directories(BEFORE ..)

if ("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
	# Supress warnings for qt headers for clang+ccache
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-inconsistent-missing-override")
endif ()

find_package (Qt5WebEngine QUIET)
qt5_add_resources(UI_RESOURCES res.qrc qml.qrc)

file(GLOB HEADERS "*.h")

set(EXECUTABLE mix)

if ("${Qt5WebEngine_VERSION_STRING}" VERSION_GREATER "5.3.0")
	set (ETH_HAVE_WEBENGINE TRUE)
	qt5_add_resources(UI_RESOURCES web.qrc)
else()
	qt5_add_resources(UI_RESOURCES noweb.qrc)
endif()

if (CMAKE_BUILD_TYPE MATCHES Debug)
	add_definitions(-DQT_QML_DEBUG)
endif()

# eth_add_executable is defined in cmake/EthExecutableHelper.cmake
eth_add_executable(${EXECUTABLE} 
	ICON mix
	UI_RESOURCES ${UI_RESOURCES}
)


set(LIBRARIES "Qt5::Core;Qt5::Gui;Qt5::Widgets;Qt5::Network;Qt5::Quick;Qt5::Qml;webthree;ethereum;evm;ethcore;devcrypto;solidity;evmcore;devcore;jsqrc;web3jsonrpc")
if (${ETH_HAVE_WEBENGINE})
	add_definitions(-DETH_HAVE_WEBENGINE)
	list(APPEND LIBRARIES "Qt5::WebEngine")
endif()

target_link_libraries(${EXECUTABLE} ${LIBRARIES})

# eth_install_executable is defined in cmake/EthExecutableHelper.cmake
eth_install_executable(${EXECUTABLE}
	QMLDIR ${CMAKE_CURRENT_SOURCE_DIR}/qml
)

#add qml asnd stdc files to project tree in Qt creator
file(GLOB_RECURSE QMLFILES "qml/*.*")
file(GLOB_RECURSE TESTFILES "test/qml/*.*")
file(GLOB_RECURSE SOLFILES "stdc/*.*")
add_custom_target(mix_qml SOURCES ${QMLFILES} ${SOLFILES} ${TESTFILES})

#test target
find_package(Qt5QuickTest REQUIRED)
find_package(Qt5Test REQUIRED)
set(TEST_EXECUTABLE mix_test)
list(APPEND LIBRARIES "Qt5::QuickTest")
list(APPEND LIBRARIES "Qt5::Test")
list(REMOVE_ITEM SRC_LIST "./main.cpp")
aux_source_directory(test SRC_LIST)
file(GLOB HEADERS "test/*.h")
add_executable(${TEST_EXECUTABLE} ${UI_RESOURCES} ${SRC_LIST} ${HEADERS})
target_link_libraries(${TEST_EXECUTABLE} ${LIBRARIES})
set_target_properties(${TEST_EXECUTABLE} PROPERTIES EXCLUDE_FROM_ALL 1 EXCLUDE_FROM_DEFAULT_BUILD 1)



