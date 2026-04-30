cmake_minimum_required(VERSION 3.13)
project(pico_w_keypad_led C CXX)

# Documentation-oriented repository scaffold.
# The source in src/main.cpp uses Arduino-style APIs (setup/loop + Keypad library)
# and is intended for Arduino-Pico or Wokwi Arduino simulation.

add_custom_target(source_only ALL
  COMMAND ${CMAKE_COMMAND} -E echo "This project is documented for Arduino-style Pico W firmware."
  SOURCES src/main.cpp
)
