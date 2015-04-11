# This code is part of MaNGOS. Contributor & Copyright details are in AUTHORS/THANKS.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA

set(EXECUTABLE_NAME ahbotd)

set(EXECUTABLE_SRCS AuctionHouseBot.cpp Main.cpp)

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/ahbot
  ${CMAKE_CURRENT_SOURCE_DIR}/playerbot
  ${CMAKE_SOURCE_DIR}/src/game
  ${CMAKE_SOURCE_DIR}/src/game/AuctionHouseBot
  ${CMAKE_SOURCE_DIR}/src/game/BattleGround
  ${CMAKE_SOURCE_DIR}/src/game/ChatCommands
  ${CMAKE_SOURCE_DIR}/src/game/Maps
  ${CMAKE_SOURCE_DIR}/src/game/MotionGenerators
  ${CMAKE_SOURCE_DIR}/src/game/movement
  ${CMAKE_SOURCE_DIR}/src/game/Object
  ${CMAKE_SOURCE_DIR}/src/game/OutdoorPvP
  ${CMAKE_SOURCE_DIR}/src/game/References
  ${CMAKE_SOURCE_DIR}/src/game/Server
  ${CMAKE_SOURCE_DIR}/src/game/Tools
  ${CMAKE_SOURCE_DIR}/src/game/vmap
  ${CMAKE_SOURCE_DIR}/src/game/WorldHandlers
  ${CMAKE_SOURCE_DIR}/src/shared
  ${CMAKE_SOURCE_DIR}/src/shared/Config
  ${CMAKE_SOURCE_DIR}/src/shared/Common
  ${CMAKE_SOURCE_DIR}/src/shared/Database
  ${CMAKE_SOURCE_DIR}/src/shared/DataStores
  ${CMAKE_SOURCE_DIR}/src/shared/Log
  ${CMAKE_SOURCE_DIR}/src/shared/Threading
  ${CMAKE_SOURCE_DIR}/src/shared/Utilities
  ${CMAKE_SOURCE_DIR}/src/framework
  ${CMAKE_SOURCE_DIR}/src/framework/Platform
  ${CMAKE_BINARY_DIR}
  ${CMAKE_BINARY_DIR}/src/shared
  ${MYSQL_INCLUDE_DIR}
  ${ACE_INCLUDE_DIR}
)

# if(PCH)
#   include_directories(${CMAKE_CURRENT_BINARY_DIR})
# endif()

add_executable(${EXECUTABLE_NAME} ${EXECUTABLE_SRCS})

if(NOT ACE_USE_EXTERNAL)
    add_dependencies(${EXECUTABLE_NAME} ace)
    target_link_libraries(${EXECUTABLE_NAME} ace)
else()
    target_link_libraries(${EXECUTABLE_NAME} ACE)
endif()

target_link_libraries(${EXECUTABLE_NAME}
    game
    shared
    framework
    g3dlite
    ${OPENSSL_LIBRARIES}
)

if(POSTGRESQL)
    include_directories(${PostgreSQL_INCLUDE_DIR})
    target_link_libraries(${EXECUTABLE_NAME} ${PostgreSQL_LIBRARIES})
else()
    include_directories(${MYSQL_INCLUDE_DIR})
    target_link_libraries(${EXECUTABLE_NAME} ${MYSQL_LIBRARY})
endif()

add_dependencies(${EXECUTABLE_NAME} game)


if(UNIX)
    target_link_libraries(${EXECUTABLE_NAME}
        ${OPENSSL_EXTRA_LIBRARIES}
        ${ZLIB_LIBRARIES}
    )
endif()


set(EXECUTABLE_LINK_FLAGS "")

if(UNIX)
    if(CMAKE_SYSTEM_NAME MATCHES "FreeBSD")
        set(EXECUTABLE_LINK_FLAGS "-Wl,--no-as-needed -pthread -lrt ${EXECUTABLE_LINK_FLAGS} -rdynamic")
    elseif(CMAKE_SYSTEM_NAME MATCHES "Linux")
        set(EXECUTABLE_LINK_FLAGS "-Wl,--no-as-needed -ldl -pthread -lrt ${EXECUTABLE_LINK_FLAGS} -rdynamic")
    endif()
endif()

if(APPLE)
    set(EXECUTABLE_LINK_FLAGS "-framework Carbon ${EXECUTABLE_LINK_FLAGS}")
endif()

set_target_properties(${EXECUTABLE_NAME} PROPERTIES LINK_FLAGS "${EXECUTABLE_LINK_FLAGS}")

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/ahbot.conf.dist.in" "${CMAKE_CURRENT_BINARY_DIR}/ahbot.conf.dist")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/ahbot.conf.dist" DESTINATION "${CONF_INSTALL_DIR}")
install(TARGETS ${EXECUTABLE_NAME} DESTINATION "${BIN_DIR}")

if(WIN32 AND MSVC)
  install(FILES "${CMAKE_CURRENT_BINARY_DIR}/\${BUILD_TYPE}/${EXECUTABLE_NAME}.pdb" DESTINATION "${BIN_DIR}" CONFIGURATIONS Debug)
  add_custom_command(TARGET ${EXECUTABLE_NAME}
    POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy "${CMAKE_CURRENT_BINARY_DIR}/ahbot.conf.dist" "${CONF_COPY_DIR}"
  )
endif()
