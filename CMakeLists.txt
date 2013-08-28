project(coman-wbi)
cmake_minimum_required(VERSION 2.8)

if(COMMAND cmake_policy)
    cmake_policy(SET CMP0003 NEW)
    cmake_policy(SET CMP0005 NEW)
endif(COMMAND cmake_policy)

#fix this to Matlab found folder
set(MATLAB_PATH "/usr/local/MATLAB/R2013a" CACHE PATH "Matlab installation path")

# internal variables

set(MBS_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(MSYSPAD_DIR "./robotran")
set(COMAN_PRJ_NAME "CoMan_LegsCad_23Dof_V4_FB")
set(MBS_SYMBOLICR "${MBS_DIR}/${COMAN_PRJ_NAME}/symbolicR")
set(MSYSPAD_SOURCE_C "${MSYSPAD_DIR}")
set(MSYSPAD_SRC_USER "${MBS_DIR}/${COMAN_PRJ_NAME}")
set(COMAN_PRJ_DIR "${MBS_DIR}/${COMAN_PRJ_NAME}/")

set(SRCS_COMAN_SENSOR   "${MSYSPAD_SOURCE_C}/MBSsensorStruct.c"
                        "${MSYSPAD_SOURCE_C}/MBSdataStruct.c"
                        "${MSYSPAD_SOURCE_C}/MBSDataStruct_xml.c"
                        "${MSYSPAD_SOURCE_C}/mbs_xml_utilities.c"
                        "${MSYSPAD_SRC_USER}/${SRCS_COMAN_USER_file}/UserModelStruct.c"
                        "${MSYSPAD_SRC_USER}/${SRCS_COMAN_USER_file}/user_sf_IO.c"
                        "${MBS_SYMBOLICR}/mbs_sensor_${COMAN_PRJ_NAME}.c"
                        "${MBS_SYMBOLICR}/mbs_dirdyna_${COMAN_PRJ_NAME}.c")

find_package(LibXml2 REQUIRED)
find_package(Armadillo REQUIRED)
find_package(LAPACK REQUIRED)
find_package(BLAS REQUIRED)
find_package(Boost COMPONENTS system filesystem REQUIRED)



add_definitions( -DCMEX
                 -DMBS_XML_INPUT
                 -DARMA_USE_LAPACK)

# TODO: is this really required?
find_package(Matlab REQUIRED)

include_directories(${Boost_INCLUDE_DIR}
                    ${LIBXML2_INCLUDE_DIR}
                    ${ARMADILLO_INCLUDE_DIRS}
                    "${MATLAB_PATH}/simulink/include"
                    "${MATLAB_PATH}/extern/include"
                    ${MSYSPAD_SOURCE_C}
                    ${MBS_SYMBOLICR}
                    ${MSYSPAD_SRC_USER}
                    ${JNI_INCLUDE_DIRS})


link_directories(${Boost_LIBRARY_DIR}
                 ${CMAKE_BINARY_DIR}
                 "${MATLAB_PATH}/bin/glnxa64")

add_library(CoMaN_2d_5R	SHARED CoMaN_2d_5R.cpp
			${SRCS_COMAN_SENSOR})

set(CMAKE_CXX_FLAGS "-fpermissive")
SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${LAPACK_LINKER_FLAGS} ${BLAS_LINKER_FLAGS}")
target_link_libraries(CoMaN_2d_5R
                      ${Boost_LIBRARIES}
                      ${LIBXML2_LIBRARIES}
                      ${ARMADILLO_LIBRARIES}
                      ${LAPACK_LIBRARIES}
                      ${BLAS_LIBRARIES}
                      mx mex mat m)
# if mex mat and mx required? CHECK

# copy project data files
ADD_CUSTOM_COMMAND( TARGET CoMaN_2d_5R
                    POST_BUILD COMMAND
                    ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/robot")
ADD_CUSTOM_COMMAND( TARGET CoMaN_2d_5R
                    POST_BUILD COMMAND
                    ${CMAKE_COMMAND} -E copy "${CMAKE_CURRENT_SOURCE_DIR}/${COMAN_PRJ_NAME}/dataR/${COMAN_PRJ_NAME}.mbsdata" "${CMAKE_BINARY_DIR}/robot/robot.mbsdata")
ADD_CUSTOM_COMMAND( TARGET CoMaN_2d_5R
                    POST_BUILD COMMAND
                    ${CMAKE_COMMAND} -E copy "${CMAKE_CURRENT_SOURCE_DIR}/${COMAN_PRJ_NAME}/dataR/${COMAN_PRJ_NAME}.mbs" "${CMAKE_BINARY_DIR}/robot/robot.mbs")
