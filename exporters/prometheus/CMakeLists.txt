# Copyright 2020, OpenTelemetry Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

include_directories(include)
if(NOT TARGET prometheus-cpp::core)
  find_package(prometheus-cpp CONFIG REQUIRED)
endif()

add_library(opentelemetry_exporter_prometheus src/exporter.cc src/collector.cc
                                              src/exporter_utils.cc)
target_include_directories(
  opentelemetry_exporter_prometheus
  PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>"
         "$<INSTALL_INTERFACE:include>")

set(PROMETHEUS_EXPORTER_TARGETS opentelemetry_exporter_prometheus)
if(TARGET pull)
  list(APPEND PROMETHEUS_EXPORTER_TARGETS pull)
endif()
if(TARGET core)
  list(APPEND PROMETHEUS_EXPORTER_TARGETS core)
endif()
target_link_libraries(
  opentelemetry_exporter_prometheus
  PUBLIC opentelemetry_metrics prometheus-cpp::pull prometheus-cpp::core)
install(
  TARGETS ${PROMETHEUS_EXPORTER_TARGETS}
  EXPORT "${PROJECT_NAME}-target"
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(
  DIRECTORY include/opentelemetry/exporters/prometheus
  DESTINATION include/opentelemetry/exporters/
  FILES_MATCHING
  PATTERN "*.h")

if(BUILD_TESTING)
  add_subdirectory(test)
endif()
