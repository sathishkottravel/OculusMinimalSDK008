project (OculusVRKernel)

set(INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/Src"
				 "${CMAKE_CURRENT_SOURCE_DIR}/Src/Kernel"
				 "${CMAKE_CURRENT_SOURCE_DIR}/Src/Util"
				 "${CMAKE_CURRENT_SOURCE_DIR}/Src/Tracing"
				 "${CMAKE_CURRENT_SOURCE_DIR}/Src/GL")

set(OCULUSVRKERNEL_INCLUDE_DIRS "${INCLUDE_DIRS}" CACHE STRING "OculusVRKernel Includes")

mark_as_advanced(OCULUSVRKERNEL_INCLUDE_DIRS)

include_directories("${OCULUSVRKERNEL_INCLUDE_DIRS}")

file(GLOB_RECURSE SOURCE_FILES Src/*.cpp Src/*.h Include/*.h)

file(GLOB_RECURSE WIN32_FILES Src/OVR_Win32_*.cpp Src/OVR_Win32_*.h Src/Kernel/OVR_ThreadsWinAPI.cpp *D3D*.cpp)
file(GLOB_RECURSE D3DX_FILES Src/*D3D*.cpp Src/*D3D*.h)
file(GLOB_RECURSE OSX_FILES Src/OVR_OSX_*.cpp Src/OVR_OSX_*.h Src/*.mm)
file(GLOB_RECURSE LINUX_FILES Src/OVR_Linux_*.cpp Src/OVR_Linux_*.h)
file(GLOB_RECURSE UNIX_FILES Src/OVR_Unix_*.cpp Src/OVR_Unix_*.h Src/Kernel/OVR_ThreadsPthread.cpp)

foreach(f ${OSX_FILES}) 
    list(REMOVE_ITEM SOURCE_FILES ${f})
endforeach(f) 

foreach(f ${UNIX_FILES}) 
    list(REMOVE_ITEM SOURCE_FILES ${f})
endforeach(f) 

foreach(f ${LINUX_FILES}) 
    list(REMOVE_ITEM SOURCE_FILES ${f})
endforeach(f) 

foreach(f ${WIN32_FILES}) 
    list(REMOVE_ITEM SOURCE_FILES ${f})
endforeach(f) 

if(WIN32)

    # Extra source
    list(APPEND SOURCE_FILES ${WIN32_FILES})
    foreach(f ${D3DX_FILES}) 
      list(REMOVE_ITEM SOURCE_FILES ${f})
    endforeach(f) 


    # Extra libs
    list(APPEND EXTRA_LIBS setupapi winmm ws2_32)

elseif(APPLE)

else()

endif()

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DOVR_BUILD_DEBUG")
add_definitions(-DOVR_ENABLE_DEVELOPER_SEARCH)
add_definitions(-DOVR_DLL_BUILD)

IF(${BUILD_SHARED_LIBS})
	add_library(OculusVRKernel SHARED ${SOURCE_FILES} ${PLATFORM_SOURCE_FILES})
ELSE()
	add_library(OculusVRKernel STATIC ${SOURCE_FILES} ${PLATFORM_SOURCE_FILES})
ENDIF()

target_link_libraries(OculusVRKernel ${EXTRA_LIBS})
set(OVR_KERNEL_LIBRARIES ${EXTRA_LIBS} CACHE STRING "Dependencies of OculusVRKernel")
