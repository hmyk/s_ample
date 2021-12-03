#cmake3.0以上
cmake_minimum_required(VERSION 3.0)

#C++プロジェクト宣言
set(project "SAMPLE")
project(${project} C CXX)

#VCのフィルタリング
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

#インストール先のディレクトリを指定するときの標準的な値を決めてくれる。
include(GNUInstallDirs)

include(CMakeDependentOption)
cmake_dependent_option(USE_MSVC_RUNTIME_LIBRARY_DLL 
			"Use MSVC runtime library DLL" ON
                     	"MSVC" OFF)

#--------------------------------------------------------------------
# Set compiler specific flags
#--------------------------------------------------------------------
if (MSVC)
  #subsystemをwindowsにするとwinアプリ consoleにするとコンソール
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /SUBSYSTEM:CONSOLE")
  if (NOT USE_MSVC_RUNTIME_LIBRARY_DLL)
    set(CompilerFlags
      CMAKE_CXX_FLAGS
      CMAKE_CXX_FLAGS_DEBUG
      CMAKE_CXX_FLAGS_RELEASE
      CMAKE_C_FLAGS
      CMAKE_C_FLAGS_DEBUG
      CMAKE_C_FLAGS_RELEASE
    )
    foreach(CompilerFlag ${CompilerFlags})
      string(REPLACE "/MD" "/MT" ${CompilerFlag} "${${CompilerFlag}}")
    endforeach()
  endif()
endif()

if (MSVC)
    set(CMAKE_SUPPRESS_REGENERATION true)
endif()

set (POSSIBLE_RAPIDJSON_DIR "${CMAKE_CURRENT_SOURCE_DIR}/external_header/rapidjson-1.1.0/include")
find_path(RAPIDJSON_INCLUDE_DIR rapidjson/rapidjson.h PATHS ${POSSIBLE_RAPIDJSON_DIR})
if (NOT RAPIDJSON_INCLUDE_DIR)
    set (RAPIDJSON_FILENAME ${CMAKE_CURRENT_SOURCE_DIR}/external_header/rapidjson.tgz)
    file(DOWNLOAD "https://github.com/miloyip/rapidjson/archive/v1.1.0.tar.gz" ${RAPIDJSON_FILENAME}
        SHOW_PROGRESS EXPECTED_HASH SHA256=bf7ced29704a1e696fbccf2a2b4ea068e7774fa37f6d7dd4039d0787f8bed98e)
    execute_process(COMMAND ${CMAKE_COMMAND} -E tar xvf ${RAPIDJSON_FILENAME} WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/external_header)
    find_path(RAPIDJSON_INCLUDE_DIR rapidjson/rapidjson.h PATHS ${POSSIBLE_RAPIDJSON_DIR})
endif()
include_directories(SYSTEM ${RAPIDJSON_INCLUDE_DIR})

if (MSVC)
  set(MAIN_SRC  "${CMAKE_CURRENT_SOURCE_DIR}/main.cpp")

  set(TEST      "${CMAKE_CURRENT_SOURCE_DIR}/test/test.cpp"
		"${CMAKE_CURRENT_SOURCE_DIR}/test/test.h")

  source_group("" FILES ${MAIN_SRC})
  source_group("test" FILES ${TEST})

  add_executable(${project} ${MAIN_SRC} ${TEST})

  target_include_directories(${project} PRIVATE "$<BUILD_INTERFACE:${POSSIBLE_RAPIDJSON_DIR}>"
						"$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/test>")

  #フォルダ作成
  set_target_properties(${project} ${project} PROPERTIES FOLDER ${project})
endif(MSVC)
