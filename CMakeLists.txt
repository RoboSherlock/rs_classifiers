cmake_minimum_required(VERSION 2.8.3)
#cmake_policy(NEW)
project(rs_classifiers)


find_package(simtrack_nodes QUIET)
if(simtrack_nodes_FOUND)
    message(STATUS "OPTIONAL MODULE simtrack FOUND")
    set(OPTIONAL_simtrack simtrack_nodes)
endif(simtrack_nodes_FOUND)

find_package(json_prolog QUIET)
if(json_prolog_FOUND)
    message(STATUS "OPTIONAL MODULE json_prolog FOUND")
    set(OPTIONAL_json_prolog json_prolog)
endif(json_prolog_FOUND)

#find_package(Caffe QUIET)
#find_package(aruco QUIET)
#find_package(MPI QUIET)

find_package(Boost REQUIRED COMPONENTS system thread date_time python numpy filesystem regex)
find_package(PythonLibs REQUIRED)
include_directories(${PYTHON_INCLUDE_DIRS})


if(Caffe_FOUND)
    add_definitions(-DWITH_CAFFE)
    message(STATUS "OPTIONAL MODULE Caffe FOUND")
    set(OPTIONAL_Caffe Caffe)
    add_definitions( -DCAFFE_FOUND ${Caffe_DEFINITIONS})
    include_directories(${Caffe_INCLUDE_DIRS})
endif(Caffe_FOUND)

find_package(catkin REQUIRED robosherlock
    message_generation
    message_runtime
	rapidjson_ros
    ${OPTIONAL_simtrack}
    ${OPTIONAL_json_prolog}
    )

message(STATUS " RS built with google googgles: " ${RS_WITH_GG})
message(STATUS " RS built with Caffe: " ${RS_WITH_CAFFE})

#################################################################################
### Constants for project                                                      ##
#################################################################################
set(NAMESPACE rs_classifiers)
set(TYPESYSTEM_CPP_PATH ${PROJECT_SOURCE_DIR}/include/rs_classifiers/types)
set(TYPESYSTEM_XML_PATH ${PROJECT_SOURCE_DIR}/descriptors/typesystem)
set(ANNOTATOR_PATH      ${PROJECT_SOURCE_DIR}/descriptors/annotators)
set(ENGINE_PATH         ${PROJECT_SOURCE_DIR}/descriptors/analysis_engines)

configure_file(${RS_PROJECT_CONFIG} ${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}_config.cmake)
#include(cmake/rs_addons_config.cmake)

################################################################################
## Find all include directories                                               ##
################################################################################
find_include_dirs(RS_ADDONS_INCLUDE_DIRS_LIST)

catkin_package(
    #INCLUDE_DIRS ${RS_ADDONS_INCLUDE_DIRS_LIST}
    #LIBRARIES ${RS_LIBS}
    CATKIN_DEPENDS robosherlock
    #CFG_EXTRAS ${PROJECT_NAME}_config.cmake
    )

include_directories(
    ${RapidJSON_INCLUDE_DIRS}
    ${RS_ADDONS_INCLUDE_DIRS_LIST}
    ${Boost_INCLUDE_DIRS}
    ${catkin_INCLUDE_DIRS}
    )

################################################################################
## Update analysis engines, typesystem and include all relevant files         ##
################################################################################
## generate classes from the typesystem xml files
generate_type_system(robosherlock)

#find all relevant files
find_additional_files()
###############################################################################
# Sub Projects                                                               ##
###############################################################################
#If you want to divide your projects into subprojects include the subdirectories
#each containing a CMakeLists.txt here
#add_subdirectory(src/xxx)


add_executable(featureExtractor src/tools/feature_extractor.cpp)
target_link_libraries(featureExtractor
  ${PCL_LIBRARIES}
  ${catkin_LIBRARIES}
  ${OpenCV_LIBRARIES}
  ${Boost_LIBRARIES}
  ${Caffe_LIBRARIES}
)


add_library(rs_classifiers src/classifiers/RSClassifier.cpp
                           src/classifiers/RSSVM.cpp
                           src/classifiers/RSRF.cpp
                           src/classifiers/RSKNN.cpp)
target_link_libraries(rs_classifiers ${OpenCV_LIBRARIES} ${catkin_LIBRARIES} ${Boost_LIBRARIES})


#this should be an executable
add_executable(train_classifier src/tools/classifier_trainer.cpp)
target_link_libraries(train_classifier rs_classifiers
        ${CATKIN_LIBRARIES}
        ${OpenCV_LIBRARIES}
        ${Boost_LIBRARIES})

rs_add_library(rs_svmAnnotator src/SvmAnnotator.cpp)
target_link_libraries(rs_svmAnnotator rs_classifiers
    ${catkin_LIBRARIES}
    ${OpenCV_LIBRARIES})

rs_add_library(rs_rfAnnotator src/RfAnnotator.cpp)
target_link_libraries(rs_rfAnnotator rs_classifiers
    ${catkin_LIBRARIES}
    ${OpenCV_LIBRARIES})


rs_add_library(rs_knnAnnotator src/KnnAnnotator.cpp)
target_link_libraries(rs_knnAnnotator rs_classifiers
    ${CATKIN_LIBRARIES}
    ${OpenCV_LIBRARIES})

