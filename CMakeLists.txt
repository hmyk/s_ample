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

if (MSVC)
  set(MAIN_SRC  "${CMAKE_CURRENT_SOURCE_DIR}/main.cpp")

  set(TEST      "${CMAKE_CURRENT_SOURCE_DIR}/test/test.cpp"
		"${CMAKE_CURRENT_SOURCE_DIR}/test/test.h")

  source_group("" FILES ${MAIN_SRC})
  source_group("test" FILES ${TEST})

  add_executable(${project} ${MAIN_SRC} ${TEST})

  target_include_directories(${project} PRIVATE "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/external_header>"
						"$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/test>")

  #フォルダ作成
  set_target_properties(${project} ${project} PROPERTIES FOLDER ${project})
endif(MSVC)
