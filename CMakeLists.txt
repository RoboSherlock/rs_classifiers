cmake_minimum_required(VERSION 2.8.3)
project(rs_classifiers)

find_package(catkin REQUIRED robosherlock
    message_generation
    message_runtime
    rapidjson_ros
    )

#################################################################################
### Constants for project                                                      ##
#################################################################################
set(NAMESPACE rs_classifiers)
set(TYPESYSTEM_CPP_PATH ${PROJECT_SOURCE_DIR}/include/rs_classifiers/types)
set(TYPESYSTEM_XML_PATH ${PROJECT_SOURCE_DIR}/descriptors/typesystem)
set(ANNOTATOR_PATH      ${PROJECT_SOURCE_DIR}/descriptors/annotators)
set(ENGINE_PATH         ${PROJECT_SOURCE_DIR}/descriptors/analysis_engines)

configure_file(${RS_PROJECT_CONFIG} ${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}_config.cmake)


################################################################################
## Find all include directories                                               ##
################################################################################
find_include_dirs(RS_Classifier_INCLUDE_DIRS_LIST)

catkin_package(
    CATKIN_DEPENDS robosherlock
    )

include_directories(
    ${RapidJSON_INCLUDE_DIRS}
    ${RS_Classifier_INCLUDE_DIRS_LIST}
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

