# Ex.No: 11  Mini Project 
### DATE: 18/10/2024                                                                           
### REGISTER NUMBER : 212222240049

### AIM: 
To write a python program to simulate the game using Pygame library and basic AI Techniques.

### Algorithm:
1. Initialize game: Set up Pygame, screen, clock, load images, and initialize game variables (player, enemies, score, etc.).

2. Handle input: Capture keyboard input for player movement and shooting bullets in different directions.

3. Update game state: Move bullets and enemies, spawn new enemies randomly, and check for collisions between bullets and enemies.

4. Check game conditions: Monitor collisions between enemies and the player to trigger game over, and reward survival points over time.

5. Render graphics: Draw background, player, bullets, enemies, and display the current score.

6. Game loop control: Repeat the game loop while handling game over conditions and restarting the game if needed.

### Program:
```Python
import pygame
import random

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
clock = pygame.time.Clock()

# Load images and scale them to fit the window
background_image = pygame.image.load('nightbase.jpg')
background_image = pygame.transform.scale(background_image, (WIDTH, HEIGHT))

player_image = pygame.image.load('dogegun.png')
player_image = pygame.transform.scale(player_image, (150, 75))

enemy_image = pygame.image.load('enemy.png')
enemy_image = pygame.transform.scale(enemy_image, (75, 75))

# Colors
WHITE = (255, 255, 255)

# Game settings
PLAYER_SPEED = 5
ENEMY_SPEED = 1
ENEMY_SPAWN_RATE = 40
POINTS_PER_HIT = 5
POINTS_PER_SECOND = 1
PENALTY_ON_COLLISION = 10

# Player
player_pos = [WIDTH // 2, HEIGHT // 2]
bullets = []  # List to hold all bullets
score = 0
start_time = pygame.time.get_ticks()

# Enemies
enemies = []

# Game states
game_over = False

# Font for displaying text
font = pygame.font.Font(None, 36)

def draw_text(text, color, x, y):
    """Draw text on the screen."""
    label = font.render(text, True, color)
    screen.blit(label, (x, y))

def reset_game():
    """Reset the game to the initial state."""
    global player_pos, bullets, enemies, score, game_over, start_time
    player_pos[:] = [WIDTH // 2, HEIGHT // 2]
    bullets.clear()  # Clear bullets
    enemies.clear()
    score = 0
    game_over = False
    start_time = pygame.time.get_ticks()

def draw_player():
    """Draw the player."""
    screen.blit(player_image, (player_pos[0], player_pos[1]))

def draw_bullets():
    """Draw all bullets."""
    for bullet in bullets:
        pygame.draw.circle(screen, (255, 255, 0), (bullet[0], bullet[1]), 5)

def move_bullets():
    """Move all bullets."""
    for bullet in bullets[:]:
        bullet[0] += bullet[2]
        bullet[1] += bullet[3]
        if bullet[0] < 0 or bullet[0] > WIDTH or bullet[1] < 0 or bullet[1] > HEIGHT:
            bullets.remove(bullet)

def draw_enemies():
    """Draw the enemies."""
    for enemy in enemies:
        screen.blit(enemy_image, enemy.topleft)

def move_enemies():
    """Move enemies towards the player."""
    for enemy in enemies:
        if enemy.x < player_pos[0]:
            enemy.x += ENEMY_SPEED
        elif enemy.x > player_pos[0]:
            enemy.x -= ENEMY_SPEED
        if enemy.y < player_pos[1]:
            enemy.y += ENEMY_SPEED
        elif enemy.y > player_pos[1]:
            enemy.y -= ENEMY_SPEED

def handle_collisions():
    """Handle bullet collisions with enemies."""
    global score
    for bullet in bullets[:]:
        for enemy in enemies:
            if enemy.collidepoint(bullet[0], bullet[1]):
                enemies.remove(enemy)
                bullets.remove(bullet)
                score += POINTS_PER_HIT
                break

def check_game_over():
    """Check if the player has lost."""
    global game_over, score
    player_rect = pygame.Rect(player_pos[0], player_pos[1], 60, 60)
    for enemy in enemies:
        if player_rect.colliderect(enemy):
            game_over = True
            score -= PENALTY_ON_COLLISION

def draw_game_over_screen():
    """Draw the game over screen with score."""
    draw_text("You Are Out!", WHITE, WIDTH // 2 - 80, HEIGHT // 2 - 20)
    draw_text(f"Your Score: {score}", WHITE, WIDTH // 2 - 80, HEIGHT // 2 + 20)
    draw_text("Press R to Restart", WHITE, WIDTH // 2 - 120, HEIGHT // 2 + 60)

# Main Game Loop
running = True
while running:
    # Draw background
    screen.blit(background_image, (0, 0))

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # Check if the game is over
    if game_over:
        draw_game_over_screen()
        keys = pygame.key.get_pressed()
        if keys[pygame.K_r]:
            reset_game()
        pygame.display.flip()
        continue

    keys = pygame.key.get_pressed()

    # Player Movement
    if keys[pygame.K_LEFT] and player_pos[0] > 0:
        player_pos[0] -= PLAYER_SPEED
    if keys[pygame.K_RIGHT] and player_pos[0] < WIDTH - 60:
        player_pos[0] += PLAYER_SPEED
    if keys[pygame.K_UP] and player_pos[1] > 0:
        player_pos[1] -= PLAYER_SPEED
    if keys[pygame.K_DOWN] and player_pos[1] < HEIGHT - 60:
        player_pos[1] += PLAYER_SPEED

    # Bullet shooting with direction
    if keys[pygame.K_w]:
        bullets.append([player_pos[0] + 30, player_pos[1], 0, -10])  # Up
    if keys[pygame.K_a]:
        bullets.append([player_pos[0], player_pos[1] + 30, -10, 0])  # Left
    if keys[pygame.K_s]:
        bullets.append([player_pos[0] + 30, player_pos[1] + 60, 0, 10])  # Down
    if keys[pygame.K_d]:
        bullets.append([player_pos[0] + 60, player_pos[1] + 30, 10, 0])  # Right

    # Move bullets and enemies
    move_bullets()
    move_enemies()

    # Handle bullet and enemy collisions
    handle_collisions()

    # Check time and reward points for surviving
    elapsed_time = pygame.time.get_ticks() - start_time
    if elapsed_time // 1000 > 0:  # Every second
        score += POINTS_PER_SECOND  # Reward points for survival
        start_time += 1000  # Update start time

    # Spawn enemies at random positions
    if random.randint(1, ENEMY_SPAWN_RATE) == 1:
        new_enemy = pygame.Rect(random.randint(0, WIDTH - 60), random.randint(0, HEIGHT - 60), 60, 60)
        enemies.append(new_enemy)

    # Check game over condition
    check_game_over()

    # Draw everything
    draw_player()
    draw_bullets()
    draw_enemies()

    # Draw score
    draw_text(f"Score: {score}", WHITE, 10, 10)

    # Update the display
    pygame.display.flip()

    # Frame rate control
    clock.tick(30)

# Quit Pygame
pygame.quit()
```

### Output:
![Screenshot 2024-10-18 134613](https://github.com/user-attachments/assets/9ca86df6-ae93-4b42-a974-844d582e739c)

### Result:
Thus the simple game was implemented using Pygame library and basic AI Techniques.
