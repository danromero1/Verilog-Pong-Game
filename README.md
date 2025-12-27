# Verilog Pong Game

![FPGA](https://img.shields.io/badge/FPGA-Nexys%20A7-orange?style=flat-square)
![Verilog](https://img.shields.io/badge/language-Verilog-brown?style=flat-square)
![VGA](https://img.shields.io/badge/display-VGA%20640x480-blue?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)

Hand-written Verilog implementation of Pong on a Nexys A7 FPGA with VGA display output, real-time paddle control, and hardware-based collision detection.

---

## Overview

This project implements a fully functional Pong game in Verilog, synthesized and deployed on a Nexys A7-100T FPGA. The design demonstrates real-time video signal generation, game state logic, and hardware-based collision physics running entirely in programmable logic with no CPU or software involved.

**Course:** ECE 3300L - Digital Circuit Design Verilog Lab  
**Institution:** Cal Poly Pomona  
**Date:** December 2024

---

## Features

**VGA Display**
- 640×480@60Hz VGA timing generation
- Hand-written horizontal and vertical sync signals
- RGB color output for game elements
- Pixel-perfect rendering at 25 MHz pixel clock

**Game Mechanics**
- Single-player Pong with button-controlled paddle
- Ball physics with directional velocity
- Collision detection with paddle and screen boundaries
- Border rendering with visual feedback

**Hardware Implementation**
- Fully synthesized digital logic (no soft processor)
- Real-time updates synchronized to VGA refresh (60 FPS)
- Debounced button inputs for paddle control
- Modular Verilog design

---

## Technical Implementation

### VGA Signal Generation

**Module:** `hvsync_generator`

Generates VGA-compatible horizontal and vertical sync signals for 640×480@60Hz display mode.

**Key Specifications:**
- Pixel Clock: 25 MHz (derived from 100 MHz board clock)
- Horizontal Sync: 31.5 kHz line frequency
- Vertical Sync: 60 Hz frame rate
- Active Display Area: 640×480 pixels

**Timing Counters:**
- `CounterX` (10-bit): Horizontal pixel position (0-799)
- `CounterY` (9-bit): Vertical line position (0-524)
- `inDisplayArea`: High when rendering visible pixels

### Paddle Control

**Input:** Push buttons (btn_up, btn_down)  
**Position Register:** 9-bit paddle position (vertical axis)

The paddle position updates on each clock cycle based on button state, with overflow/underflow protection to keep it within the display boundaries.

```verilog
always @(posedge clk) begin
    if (btn_up && ~&PaddlePosition)
        PaddlePosition <= PaddlePosition + 1;
    else if (btn_down && |PaddlePosition)
        PaddlePosition <= PaddlePosition - 1;
end
```

### Ball Physics

**Position Registers:**
- `ballX` (10-bit): Horizontal position
- `ballY` (9-bit): Vertical position

**Direction Registers:**
- `ball_dirX`: Horizontal direction (0=right, 1=left)
- `ball_dirY`: Vertical direction (0=down, 1=up)

The ball position updates once per frame (60 times per second), changing direction upon collision with walls or paddle.

### Collision Detection

**Collision Zones:**
- `CollisionX1`: Left side of ball
- `CollisionX2`: Right side of ball  
- `CollisionY1`: Top of ball
- `CollisionY2`: Bottom of ball

Collision flags are set when the ball overlaps with the paddle or screen borders, triggering direction changes.

**Bouncing Objects:**
- Screen borders (top, bottom, left, right)
- Paddle (horizontal bar controlled by player)

### Graphics Rendering

**RGB Output Logic:**
```verilog
wire R = BouncingObject | ball | (CounterX[3] ^ CounterY[3]);
wire G = BouncingObject | ball;
wire B = BouncingObject | ball;
```

- **Ball:** White square (RGB all high)
- **Paddle:** White rectangle
- **Borders:** White lines
- **Background:** Checkerboard pattern (alternating tiles)

---

## Hardware Configuration

**FPGA Board:** Digilent Nexys A7-100T  
**Xilinx Device:** XC7A100T-1CSG324C  
**Clock:** 100 MHz onboard oscillator

**Pin Assignments (from constraints file):**

| Signal | FPGA Pin | Description |
|--------|----------|-------------|
| clk | E3 | 100 MHz system clock |
| btn_up | M18 | Paddle up button |
| btn_down | P18 | Paddle down button |
| vga_R | A3 | VGA red output |
| vga_G | C6 | VGA green output |
| vga_B | B7 | VGA blue output |
| vga_h_sync | B11 | VGA horizontal sync |
| vga_v_sync | B12 | VGA vertical sync |

---

## Project Structure

```
Verilog-Pong-Game/
├── src/
│   ├── hvsync_generator.v    # VGA timing generator
│   └── pong.v                 # Main game logic and top module
├── Constraints/
│   └── Nexys-A7-100T-Master_Project.xdc  # Pin constraints
└── README.md
```

---

## Implementation Details

**Design Methodology:**
1. VGA sync signal generation with proper timing
2. Pixel counters for screen position tracking
3. Game state registers (paddle position, ball position/direction)
4. Combinational collision detection logic
5. Sequential state updates synchronized to frame refresh

**Key Challenges:**
- Generating stable VGA timing from 100 MHz clock
- Implementing collision detection in hardware logic
- Synchronizing game updates to 60 Hz frame rate
- Managing multiple concurrent signal updates

**Learning Outcomes:**
- VGA protocol implementation and timing requirements
- Finite state machine design for game logic
- Real-time hardware constraints and synchronization
- Verilog coding for synthesis (not just simulation)
- FPGA toolchain workflow (synthesis, implementation, bitstream generation)

---

## Building and Running

**Requirements:**
- Xilinx Vivado (tested with 2020.2 or later)
- Nexys A7-100T FPGA board
- VGA monitor and cable
- USB cable for programming

**Steps:**

1. **Create Vivado Project**
   ```
   - New Project → RTL Project
   - Add source files (hvsync_generator.v, pong.v)
   - Add constraints file (.xdc)
   - Select board: Nexys A7-100T
   ```

2. **Synthesize and Implement**
   ```
   - Run Synthesis
   - Run Implementation  
   - Generate Bitstream
   ```

3. **Program FPGA**
   ```
   - Open Hardware Manager
   - Connect to Nexys A7 via USB
   - Program device with generated .bit file
   ```

4. **Connect VGA Display**
   ```
   - Connect VGA cable from FPGA to monitor
   - Monitor should auto-detect 640×480@60Hz signal
   ```

5. **Play**
   ```
   - Use btn_up (BTNU) and btn_down (BTND) to control paddle
   - Ball bounces automatically
   ```

---

## Technical Specifications

**Display Resolution:** 640×480 pixels  
**Refresh Rate:** 60 Hz  
**Color Depth:** 1-bit per channel (8 colors possible)  
**Ball Size:** 16×16 pixels  
**Paddle Size:** ~112×16 pixels  
**Update Rate:** 60 FPS (synchronized to VGA frame)

**Resource Utilization** (Nexys A7-100T):
- Logic cells: <1% (minimal logic)
- Flip-flops: ~50 registers
- LUTs: ~100 combinational elements
- No block RAM or DSP slices required

---

## Module Descriptions

### hvsync_generator

**Purpose:** Generate VGA horizontal and vertical sync signals

**Inputs:**
- `clk`: 25 MHz pixel clock (or 100 MHz with clock divider)

**Outputs:**
- `vga_h_sync`: Horizontal sync signal
- `vga_v_sync`: Vertical sync signal
- `inDisplayArea`: High during active video region
- `CounterX`: Current horizontal pixel position
- `CounterY`: Current vertical line position

### pong (Top Module)

**Purpose:** Main game logic and graphics controller

**Inputs:**
- `clk`: System clock (100 MHz)
- `btn_up`: Button to move paddle up
- `btn_down`: Button to move paddle down

**Outputs:**
- `vga_h_sync`: Horizontal sync to VGA
- `vga_v_sync`: Vertical sync to VGA  
- `vga_R`: Red color channel
- `vga_G`: Green color channel
- `vga_B`: Blue color channel

---

## Future Enhancements

**Potential Improvements:**
- Score tracking with 7-segment display
- Two-player mode with second paddle
- Variable ball speed (increase over time)
- Sound effects using PWM audio output
- AI opponent with paddle tracking logic
- Multiple difficulty levels
- Game reset button
- Pause functionality

---

## References

**VGA Timing:**
- VESA VGA 640×480@60Hz Timing Standard
- [tinyvga.com/vga-timing](http://tinyvga.com/vga-timing)

**FPGA Resources:**
- Nexys A7 Reference Manual (Digilent)
- Xilinx 7-Series FPGA User Guide
- Vivado Design Suite Documentation

**Course Materials:**
- ECE 3300L Lab Manual (Cal Poly Pomona)
- Digital Design and Computer Architecture (Harris & Harris)

---

## Acknowledgments

**Course:** ECE 3300L - Digital Circuit Design Verilog Lab  
**Instructor:** Prof. Daniel Van Blerkom  
**Partner:** Isaias Rafael-Sepulveda  
**Institution:** California Polytechnic State University, Pomona

This project was completed as the final assignment for ECE 3300L, demonstrating practical application of Verilog HDL for real-time video game implementation on FPGA hardware.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Contact

**Daniel Romero**  
Email: daniel.romero@ieee.org  
Portfolio: [electricalromero.com](https://electricalromero.com)  
LinkedIn: [linkedin.com/in/daniel-romero-ee](https://www.linkedin.com/in/daniel-romero-ee/)

---

*A hardware-based Pong implementation showcasing FPGA digital design, VGA interfacing, and real-time game logic in Verilog.*

*Last Updated: December 2025*
