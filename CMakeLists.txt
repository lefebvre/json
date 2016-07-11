TRIBITS_PACKAGE(JSON)

INCLUDE_DIRECTORIES("${CMAKE_CURRENT_SOURCE_DIR}")

FIND_PROGRAM(FLEX_PATH NAME flex HINTS ENV PATH)
SET(FLEX_OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/lex.yy.cc")
IF(FLEX_PATH STREQUAL "FLEX_PATH-NOTFOUND")
   MESSAGE(WARNING "Flex not found. Generating/updating lexer disabled.")
ELSE()
   MESSAGE(STATUS "Found flex : '${FLEX_PATH}'")
   ADD_CUSTOM_TARGET(json_lexer
       COMMAND ${FLEX_PATH}
               --outfile=${FLEX_OUTPUT}
               ${CMAKE_CURRENT_SOURCE_DIR}/json.l
       WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
       )
ENDIF()
FIND_PROGRAM(BISON_PATH NAME bison HINTS ENV PATH)
SET(BISON_OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/json.tab.cc")
IF(BISON_PATH STREQUAL "BISON_PATH-NOTFOUND")
   MESSAGE(WARNING "Bison not found. Generating/updating parser disabled.")
ELSE()
   SET(BISON_DEFINES "${CMAKE_CURRENT_SOURCE_DIR}/json.tab.hh")
   MESSAGE(STATUS "Found bison : '${BISON_PATH}'")
   ADD_CUSTOM_TARGET(json_parser
       COMMAND ${BISON_PATH}
               --defines=${BISON_DEFINES}
               --output=${BISON_OUTPUT}
               ${CMAKE_CURRENT_SOURCE_DIR}/json.y
       WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
       DEPENDS json_lexer)
ENDIF()

SET(HEADERS
   json.hh
   json_st.hh
   json.tab.hh
   json.l
   json.y
)
TRIBITS_ADD_LIBRARY(json
   SOURCES json_st.cc ${BISON_OUTPUT} ${FLEX_OUTPUT}
   NOINSTALLHEADERS ${HEADERS}
)

# Install headers into include subdirectory
INSTALL(FILES ${HEADERS} DESTINATION "include/json")

FIND_PACKAGE(Doxygen)
IF(DOXYGEN_FOUND)
        IF(DOT)
          SET(HAVE_DOT YES)
        ELSE()
          SET(HAVE_DOT NO)
        ENDIF()
        CONFIGURE_FILE(
                ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
                @ONLY
        )
        ADD_CUSTOM_TARGET(
                doc ALL
                ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
                WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
                COMMENT "Generating API documentation with Doxygen" VERBATIM
        )
ENDIF(DOXYGEN_FOUND)
TRIBITS_PACKAGE_POSTPROCESS()
