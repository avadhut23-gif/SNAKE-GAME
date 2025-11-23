# SNAKE-GAME

# Snake Game in Python using Pygame
# Works on Replit. Make sure to add "pygame" in the Packages.
import pygame
import sys
import random

# --- Game config ---
WIDTH, HEIGHT = 600, 400
CELL_SIZE = 20
GRID_COLS = WIDTH // CELL_SIZE
GRID_ROWS = HEIGHT // CELL_SIZE

FPS = 12  # Increase for faster snake

# Colors
BLACK = (0, 0, 0)
GRAY = (40, 40, 40)
GREEN = (0, 200, 0)
RED = (200, 0, 0)
WHITE = (255, 255, 255)
YELLOW = (255, 200, 0)

# Directions
UP = (0, -1)
DOWN = (0, 1)
LEFT = (-1, 0)
RIGHT = (1, 0)

def draw_grid(surface):
    for x in range(0, WIDTH, CELL_SIZE):
        pygame.draw.line(surface, GRAY, (x, 0), (x, HEIGHT))
    for y in range(0, HEIGHT, CELL_SIZE):
        pygame.draw.line(surface, GRAY, (0, y), (WIDTH, y))

def rand_cell(exclude):
    while True:
        pos = (random.randint(0, GRID_COLS - 1), random.randint(0, GRID_ROWS - 1))
        if pos not in exclude:
            return pos

def draw_cell(surface, pos, color):
    x, y = pos
    rect = pygame.Rect(x * CELL_SIZE, y * CELL_SIZE, CELL_SIZE, CELL_SIZE)
    pygame.draw.rect(surface, color, rect)

def show_text(surface, text, size, color, center):
    font = pygame.font.SysFont("consolas", size)
    surf = font.render(text, True, color)
    rect = surf.get_rect(center=center)
    surface.blit(surf, rect)

def main():
    pygame.init()
    screen = pygame.display.set_mode((WIDTH, HEIGHT))
    pygame.display.set_caption("Snake - Replit/Pygame")
    clock = pygame.time.Clock()

    # Game state
    snake = [(GRID_COLS // 2, GRID_ROWS // 2)]
    direction = RIGHT
    pending_dir = RIGHT  # buffer to avoid reversing immediately
    food = rand_cell(set(snake))
    score = 0
    high_score = 0
    game_over = False
    paused = False

    while True:
        # --- Events ---
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                if event.key in (pygame.K_UP, pygame.K_w):
                    if direction != DOWN:
                        pending_dir = UP
                elif event.key in (pygame.K_DOWN, pygame.K_s):
                    if direction != UP:
                        pending_dir = DOWN
                elif event.key in (pygame.K_LEFT, pygame.K_a):
                    if direction != RIGHT:
                        pending_dir = LEFT
                elif event.key in (pygame.K_RIGHT, pygame.K_d):
                    if direction != LEFT:
                        pending_dir = RIGHT
                elif event.key == pygame.K_p:
                    paused = not paused
                elif event.key == pygame.K_r and game_over:
                    # Reset
                    snake = [(GRID_COLS // 2, GRID_ROWS // 2)]
                    direction = RIGHT
                    pending_dir = RIGHT
                    food = rand_cell(set(snake))
                    score = 0
                    game_over = False
                    paused = False

        if paused:
            screen.fill(BLACK)
            draw_grid(screen)
            show_text(screen, "Paused (P to resume)", 28, YELLOW, (WIDTH // 2, HEIGHT // 2))
            pygame.display.flip()
            clock.tick(6)
            continue

        if not game_over:
            # Update direction once per tick
            direction = pending_dir

            # Move snake
            head_x, head_y = snake[0]
            dx, dy = direction
            new_head = (head_x + dx, head_y + dy)

            # Check wall collision
            if not (0 <= new_head[0] < GRID_COLS and 0 <= new_head[1] < GRID_ROWS):
                game_over = True
            # Check self collision
            elif new_head in snake:
                game_over = True
            else:
                snake.insert(0, new_head)
                # Eat food?
                if new_head == food:
                    score += 1
                    food = rand_cell(set(snake))
                    high_score = max(high_score, score)
                else:
                    snake.pop()

        # --- Draw ---
        screen.fill(BLACK)
        draw_grid(screen)
        # Draw food
        draw_cell(screen, food, RED)
        # Draw snake
        for i, segment in enumerate(snake):
            color = GREEN if i > 0 else YELLOW  # head highlighted
            draw_cell(screen, segment, color)

        # HUD
        show_text(screen, f"Score: {score}  High: {high_score}", 22, WHITE, (110, 16))

        if game_over:
            show_text(screen, "Game Over", 42, RED, (WIDTH // 2, HEIGHT // 2 - 20))
            show_text(screen, "Press R to restart", 24, WHITE, (WIDTH // 2, HEIGHT // 2 + 20))

        pygame.display.flip()
        clock.tick(FPS)

if __name__ == "__main__":
    main()
