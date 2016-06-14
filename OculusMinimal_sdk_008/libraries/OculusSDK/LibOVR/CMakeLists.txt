project (OculusVR)

set(INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/Include" 
				 "${CMAKE_CURRENT_SOURCE_DIR}/Src/Kernel"
				 "${CMAKE_CURRENT_SOURCE_DIR}/Src/Util"
				 "${CMAKE_CURRENT_SOURCE_DIR}/../3rdParty/glext")

set(OCULUSVR_INCLUDE_DIRS "${INCLUDE_DIRS}" CACHE STRING "OculusVR Includes")

mark_as_advanced(OCULUSVR_INCLUDE_DIRS)

include_directories("${OCULUSVR_INCLUDE_DIRS}")

file(GLOB_RECURSE SOURCE_FILES Src/*.cpp Src/*.c Src/*.h Include/*.h)

file(GLOB_RECURSE WIN32_FILES Src/OVR_Win32_*.cpp Src/OVR_Win32_*.h Src/Kernel/OVR_ThreadsWinAPI.cpp *D3D*.cpp)
file(GLOB_RECURSE D3DX_FILES Src/*D3D*.cpp Src/*D3D*.h)
file(GLOB_RECURSE OSX_FILES Src/OVR_OSX_*.cpp Src/OVR_OSX_*.h Src/*.mm)
file(GLOB_RECURSE LINUX_FILES Src/OVR_Linux_*.cpp Src/OVR_Linux_*.h)
file(GLOB_RECURSE UNIX_FILES Src/OVR_Unix_*.cpp Src/OVR_Unix_*.h Src/Kernel/OVR_ThreadsPthread.cpp)

if (WIN32 OR APPLE) 
option(OVR_USE_SHIM "Create a small shim which dynamically loads the real Oculus library" 1)
else()
option(OVR_USE_SHIM "Create a small shim which dynamically loads the real Oculus library" 0)
endif()

if (NOT OVR_USE_SHIM)
	file(GLOB_RECURSE SHIM_SOURCE_FILES 
        Src/*OVR_CAPIShim*.c
        Src/*OVR_CAPI_Util*.cpp  
        Include/*OVR_CAPI_Util*.h  
        #Src/*OVR_StereoProjection.cpp
        #Include/*OVR_StereoProjection*.h
    )
	ivw_message(${SHIM_SOURCE_FILES})
endif()


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
	
	foreach(f ${SHIM_SOURCE_FILES}) 
		list(REMOVE_ITEM WIN32_FILES ${f})
		list(REMOVE_ITEM SOURCE_FILES ${f})
		ivw_message(${f})
	endforeach(f) 

    # Extra source
    list(APPEND SOURCE_FILES ${WIN32_FILES})
    foreach(f ${D3DX_FILES}) 
      list(REMOVE_ITEM SOURCE_FILES ${f})
    endforeach(f) 


    # Extra libs
    list(APPEND EXTRA_LIBS setupapi winmm ws2_32)

elseif(APPLE)

    # Extra source
    list(APPEND SOURCE_FILES ${OSX_FILES})
    list(APPEND SOURCE_FILES ${UNIX_FILES})

    # Some special files that apparently don’t belong in the SDK ?!
    file(GLOB_RECURSE OSX_PRIVATE_FILES Src/Displays/OVR_OSX_Focus*)
    foreach(f ${OSX_PRIVATE_FILES}) 
        list(REMOVE_ITEM SOURCE_FILES ${f})
    endforeach(f) 

    # Extra libs
    find_library(COCOA_LIBRARY Cocoa)
    find_library(IOKIT_LIBRARY IOKit)
    list(APPEND EXTRA_LIBS ${COCOA_LIBRARY} ${IOKIT_LIBRARY})

    find_package(OpenGL REQUIRED)
    list(APPEND EXTRA_LIBS ${OPENGL_LIBRARIES})

else()

    # Extra source
    list(APPEND SOURCE_FILES ${LINUX_FILES})
    list(APPEND SOURCE_FILES ${UNIX_FILES})

    # Extra libs
    add_subdirectory(../3rdParty/EDID ${CMAKE_CURRENT_BINARY_DIR}/../3rdParty/EDID)
    
    # Used by the SDK to read the EDID of the monitors
    list(APPEND EXTRA_LIBS edid rt)

    # Used by the SDK to access the HID functions
    list(APPEND EXTRA_LIBS udev)

    find_package(Threads REQUIRED)
    list(APPEND EXTRA_LIBS ${CMAKE_THREAD_LIBS_INIT})

    # Check for XRandR (modern resolution switching and gamma control)
    find_package(X11 REQUIRED)
    if (NOT X11_Xrandr_FOUND)
        message(FATAL_ERROR "The RandR library and headers were not found")
    endif()
    list(APPEND EXTRA_LIBS ${X11_X11_LIB}) 
    list(APPEND EXTRA_LIBS ${X11_Xrandr_LIB})

    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../3rdParty/EDID)

endif()

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DOVR_BUILD_DEBUG")
#add_definitions(-DOVR_DECLARE_IMPORT)
#add_definitions(-DOVR_DLSYM)
add_definitions(-DOVR_ENABLE_DEVELOPER_SEARCH)
add_definitions(-DOVR_DLL_BUILD)

IF(${BUILD_SHARED_LIBS})
    
	add_library(OculusVR SHARED ${SOURCE_FILES} ${PLATFORM_SOURCE_FILES})
ELSE()
	add_library(OculusVR STATIC ${SOURCE_FILES} ${PLATFORM_SOURCE_FILES})
ENDIF()

target_link_libraries(OculusVR ${EXTRA_LIBS})
set(OVR_LIBRARIES ${EXTRA_LIBS} CACHE STRING "Dependencies of OculusVR")
