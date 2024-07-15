CMAKE_MINIMUM_REQUIRED(VERSION 3.30)

project(CNPYpp LANGUAGES CXX C VERSION 2.1.1)
set(CMAKE_CXX_STANDARD 20)

add_library(cnpy++ "src/cnpy++.cpp" "src/buffer.cpp" "src/c_interface.c")

get_directory_property(hasParent PARENT_DIRECTORY)

option(CNPYPP_USE_LIBZIP "require libzip to enable support for npz" ON)

set(minimum_boost_version 1.74)

include(GNUInstallDirs)

find_package(libzip REQUIRED)
find_package(Boost ${minimum_boost_version} COMPONENTS filesystem iostreams REQUIRED)

target_compile_features(cnpy++ PUBLIC cxx_std_20)
set_property(TARGET cnpy++ PROPERTY CXX_EXTENSIONS OFF)
target_include_directories(cnpy++ PUBLIC ${Boost_INCLUDE_DIR})
target_include_directories(cnpy++ SYSTEM PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>)
target_include_directories(cnpy++ SYSTEM INTERFACE $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/include>)
target_link_libraries(cnpy++ PRIVATE Boost::filesystem Boost::iostreams)
target_link_libraries(cnpy++ PRIVATE libzip::zip)

if(MSVC)
  target_compile_options(cnpy++ PRIVATE /W4 /WX)
else()
  target_compile_options(cnpy++ PRIVATE -Wall -Wextra -pedantic -Wfatal-errors)
endif()

install(TARGETS "cnpy++" LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES "include/cnpy++/tuple_util.hpp"
    "include/cnpy++/stride_iterator.hpp"
    "include/cnpy++/map_type.hpp"
    "include/cnpy++/buffer.hpp" DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/cnpy++)
install(FILES "include/cnpy++.hpp" "include/cnpy++.h" DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

if (NOT hasParent)
  add_executable(example1 "examples/example1.cpp")
  target_link_libraries(example1 cnpy++)

  add_executable(example2 "examples/example2.cpp")
  target_link_libraries(example2 cnpy++)

  add_executable(example_c "examples/example_c.c")
  target_link_libraries(example_c cnpy++)

  add_executable(range_example "examples/range_example.cpp")
  target_link_libraries(range_example cnpy++)
  target_compile_features(range_example PRIVATE cxx_std_20)
  set_property(TARGET range_example PROPERTY CXX_EXTENSIONS OFF)

  add_executable(range_zip_example "examples/range_zip_example.cpp")
  target_link_libraries(range_zip_example cnpy++)
  target_compile_features(range_zip_example PRIVATE cxx_std_20)
  set_property(TARGET range_zip_example PROPERTY CXX_EXTENSIONS OFF)
endif()
