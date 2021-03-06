set(MRT_PKG_VERSION 2.2.1)
# Modify only if you know what you are doing!
cmake_minimum_required(VERSION 2.8.12)
project(kitaf_utils_ros)

###################
## find packages ##
###################
find_package(mrt_cmake_modules REQUIRED)
include(UseMrtStdCompilerFlags)
include(UseMrtAutoTarget)

include(GatherDeps)
# Remove libs which cannot be found automatically
#list(REMOVE_ITEM DEPENDEND_PACKAGES <package name 1> <package name 2> ...)
find_package(AutoDeps REQUIRED COMPONENTS ${DEPENDEND_PACKAGES})

# Manually resolve removed dependend packages
#find_package(...)

# Mark files or folders for display in IDEs
mrt_add_to_ide(README.md .gitlab-ci.yml msg)


########################
## add python modules ##
########################
# This adds a python module if located under src/{PROJECT_NAME)
mrt_python_module_setup()

file(GLOB PROJECT_PYTHON_SOURCE_FILES_SRC RELATIVE "${CMAKE_CURRENT_LIST_DIR}" "python_api/*.cpp")
if (PROJECT_PYTHON_SOURCE_FILES_SRC)
    # Add a cpp-python api library. Make sure there are no name collisions with python modules in this project
    mrt_add_python_api( ${PROJECT_NAME}
        FILES ${PROJECT_PYTHON_SOURCE_FILES_SRC}
        )
endif()

################################################
## Declare ROS messages, services and actions ##
################################################

# Add message, service and action files
glob_ros_files(add_message_files msg)
glob_ros_files(add_service_files srv)
glob_ros_files(add_action_files action)

# Generate added messages and services with any dependencies listed here
if (ROS_GENERATE_MESSAGES)
    generate_messages(
        DEPENDENCIES
        #add dependencies here
        std_msgs
        )
endif()

file(GLOB SRV_FILES RELATIVE "${CMAKE_CURRENT_LIST_DIR}" "srv/*.srv")

# Generate dynamic reconfigure options
file(GLOB PARAMS_FILES RELATIVE "${CMAKE_CURRENT_LIST_DIR}" "cfg/*.params" "cfg/*.cfg" "cfg/*.mrtcfg" "cfg/*.rosif")
if (PARAMS_FILES)
    generate_ros_parameter_files(${PARAMS_FILES})
    generate_ros_interface_files(${PARAMS_FILES})
endif ()

############################
## read source code files ##
############################
file(GLOB_RECURSE PROJECT_HEADER_FILES_INC RELATIVE "${CMAKE_CURRENT_LIST_DIR}" "include/*.h" "include/*.hpp")
file(GLOB PROJECT_SOURCE_FILES_INC RELATIVE "${CMAKE_CURRENT_LIST_DIR}" "src/*.h" "src/*.hpp")
file(GLOB PROJECT_SOURCE_FILES_SRC RELATIVE "${CMAKE_CURRENT_LIST_DIR}" "src/*.cpp")

if (PROJECT_SOURCE_FILES_SRC)
    set(LIBRARY_NAME ${PROJECT_NAME})
endif()

###################################
## catkin specific configuration ##
###################################
# The catkin_package macro generates cmake config files for your package
# Declare things to be passed to dependent projects
# INCLUDE_DIRS: uncomment this if you package contains header files
# LIBRARIES: libraries you create in this project that dependent projects also need
# CATKIN_DEPENDS: catkin_packages dependent projects also need
# DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
    INCLUDE_DIRS include ${mrt_EXPORT_INCLUDE_DIRS}
    LIBRARIES ${LIBRARY_NAME} ${mrt_EXPORT_LIBRARIES}
    CATKIN_DEPENDS ${catkin_EXPORT_DEPENDS}
    )

###########
## Build ##
###########
# Add include and library directories
include_directories(
    include/${PROJECT_NAME}
    ${mrt_INCLUDE_DIRS}
    ${catkin_INCLUDE_DIRS}
    )

link_directories(
    ${mrt_LIBRARY_DIRS}
    )

# Declare a cpp library
mrt_add_library(${PROJECT_NAME}
    INCLUDES ${PROJECT_HEADER_FILES_INC} ${PROJECT_SOURCE_FILES_INC} 
    SOURCES ${PROJECT_SOURCE_FILES_SRC}
    )

#############
## Install ##
#############
# Install all targets, headers by default and scripts and other files if specified (folders or files)
mrt_install(PROGRAMS scripts FILES launch rviz maps res data nodelet_plugins.xml plugin_description.xml)

#############
## Testing ##
#############
# Add test targets for cpp and python tests
if (CATKIN_ENABLE_TESTING)
    mrt_add_ros_tests(test)
    mrt_add_tests(test)
    mrt_add_nosetests(test)
endif()
