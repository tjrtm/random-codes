import machine
import neopixel
import random
import math
import time

# =========================
# CONFIG
# =========================
LED_PIN = 1             # GPIO pin for NeoPixel data
NUM_LEDS = 256          # 16x16 LED matrix
BRIGHTNESS = 0.1        # Overall brightness (0.0 to 1.0)

# =========================
# SETUP NEOPIXEL
# =========================
np = neopixel.NeoPixel(machine.Pin(LED_PIN, machine.Pin.OUT), NUM_LEDS)

def apply_brightness(color):
    """Scale an (r,g,b) by BRIGHTNESS."""
    r, g, b = color
    return (int(r * BRIGHTNESS), int(g * BRIGHTNESS), int(b * BRIGHTNESS))

# =========================
# FIRE ANIMATION PARAM SETS
# =========================
animation_normal = {
    "cooling": 40,
    "sparking": 255,
    "speed": 0.005,
    "max_flares": 18,
    "flare_rows": 2,
    "flare_decay": 80
}

animation_fat = {
    "cooling": 255,
    "sparking": 255,
    "speed": 0.001,
    "max_flares": 18,
    "flare_rows": 2,
    "flare_decay": 10
}

def fire_animation(cooling=40, sparking=255, speed=0.005, 
                   max_flares=18, flare_rows=2, flare_decay=80):
    """
    A realistic fire effect for a 16x16 LED matrix with customizable parameters.
    Runs continuously until program is stopped.
    
    Parameters:
    - cooling: How quickly each LED cools down (0-255). Higher = faster cooling
    - sparking: Chance of new sparks (0-255). Higher = more sparks
    - speed: Delay between frames in seconds
    - max_flares: Maximum number of simultaneous bright spots
    - flare_rows: Number of bottom rows that can have flares
    - flare_decay: How quickly flares fade out
    """
    print("Running FIRE animation...")
    
    MATRIX_WIDTH = 16
    MATRIX_HEIGHT = 16
    
    # Color palette from coolest to hottest
    COLORS = [
        (0, 0, 0),      # Black
        (16, 0, 0),     # Darkest red
        (48, 0, 0),
        (96, 0, 0),
        (128, 0, 0),
        (160, 0, 0),
        (192, 32, 0),   # Red with a bit of green
        (192, 64, 0),
        (192, 96, 0),
        (192, 128, 0),  # Orange
        (128, 112, 10)  # White-ish for the hottest parts
    ]
    N_COLORS = len(COLORS)
    
    # Initialize the fire array
    fire = [[0 for _ in range(MATRIX_WIDTH)] for _ in range(MATRIX_HEIGHT)]
    # Initialize bottom row with maximum heat
    for x in range(MATRIX_WIDTH):
        fire[0][x] = N_COLORS - 1
        
    # Initialize flare tracking
    flares = []  # Each flare is (x, y, intensity)

    def pos_to_index(x, y):
        """Convert x,y matrix position to LED strip index."""
        # Zigzag layout: odd rows are reversed
        if y & 1:  # Odd row
            x = MATRIX_WIDTH - 1 - x
        return y * MATRIX_WIDTH + x

    def apply_flare(px, py, intensity):
        """Apply flare effect around a point."""
        radius = intensity * 10 // flare_decay + 1
        for dy in range(-radius, radius + 1):
            for dx in range(-radius, radius + 1):
                nx, ny = px + dx, py + dy
                if 0 <= nx < MATRIX_WIDTH and 0 <= ny < MATRIX_HEIGHT:
                    dist = int(math.sqrt(dx * dx + dy * dy) * 10)
                    heat = max(0, intensity - dist // flare_decay)
                    if heat > fire[ny][nx]:
                        fire[ny][nx] = heat

    try:
        while True:
            # 1. Cool down every cell
            for y in range(MATRIX_HEIGHT):
                for x in range(MATRIX_WIDTH):
                    cooldown = random.randint(0, ((cooling * 10) // NUM_LEDS) + 2)
                    if fire[y][x] > cooldown:
                        fire[y][x] -= cooldown
                    else:
                        fire[y][x] = 0

            # 2. Heat rises and spreads
            for y in range(MATRIX_HEIGHT - 1, 1, -1):
                for x in range(MATRIX_WIDTH):
                    left = max(0, x - 1)
                    right = min(MATRIX_WIDTH - 1, x + 1)
                    below = fire[y - 1][x]
                    below_left = fire[y - 1][left]
                    below_right = fire[y - 1][right]
                    fire[y][x] = (below + below_left + below_right) // 3

            # 3. Random sparks at bottom (give each bottom cell some random heat)
            for x in range(MATRIX_WIDTH):
                if fire[0][x] > 0:
                    # This effectively keeps the bottom row near the hot end of the palette
                    fire[0][x] = random.randint(N_COLORS - 6, N_COLORS - 2)

            # 4. Handle flares
            # Update existing flares
            i = 0
            while i < len(flares):
                fx, fy, intensity = flares[i]
                apply_flare(fx, fy, intensity)
                if intensity > 1:
                    flares[i] = (fx, fy, intensity - 1)
                    i += 1
                else:
                    flares.pop(i)

            # Possibly add new flare
            if len(flares) < max_flares and random.randint(0, 255) < sparking:
                fx = random.randint(0, MATRIX_WIDTH - 1)
                fy = random.randint(0, min(flare_rows, MATRIX_HEIGHT - 1))
                intensity = N_COLORS - 1
                flares.append((fx, fy, intensity))
                apply_flare(fx, fy, intensity)

            # 5. Convert fire array to LED colors
            for y in range(MATRIX_HEIGHT):
                for x in range(MATRIX_WIDTH):
                    heat_index = min(fire[y][x], N_COLORS - 1)
                    color = COLORS[heat_index]
                    np[pos_to_index(x, y)] = apply_brightness(color)

            np.write()
            time.sleep(speed)

    except KeyboardInterrupt:
        # Clear all LEDs when program is interrupted
        for i in range(NUM_LEDS):
            np[i] = (0, 0, 0)
        np.write()
        print("\nFire animation stopped")


fire_animation(**animation_fat)

