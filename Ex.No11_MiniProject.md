### Ex.No: 11 Mini Project
### DATE:
### REGISTER NUMBER:212222240056
# Aim:
To write a Python program to simulate the Mental Health Awareness Memory Game using Pygame and Q-Learning.
# Algorithm:
1.Initialize Pygame, the screen, and game settings.

2.Load images representing mental health tips and resize them to fit the game grid.

3.Create a shuffled deck of cards with each image appearing twice for matching pairs.

4.Implement Q-Learning parameters such as learning rate (alpha), discount factor (gamma), and exploration rate (epsilon).

5.Main game loop:
   AI flips two cards using Q-learning.
   If a match is found, display the mental health tip associated with the card.
   If not, the cards are hidden again.
   
6.Update the Q-table based on whether the AI found a match or not.

7.Repeat the process until all pairs are matched.

8.Display the number of moves taken by the AI to win the game.

9.Terminate the game when all matches are found.

### Program:

```py

import pygame
import random
import numpy as np
import os

# Initialize Pygame
pygame.init()

# Screen settings
WIDTH, HEIGHT = 600, 600
ROWS, COLS = 4, 4  # Grid for 8 pairs of images
CARD_SIZE = WIDTH // COLS  # Card size is dynamically calculated

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)

# Set up the screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Mental Health Awareness Memory Game")

# Load fonts
font = pygame.font.SysFont(None, 30)
win_font = pygame.font.SysFont(None, 48)

# Load images and corresponding messages
image_folder = 'images/'
image_files = ['meditation.png', 'exercise.png', 'breathing.png', 'relaxation.png', 
               'therapy.png', 'support.png', 'positivity.png', 'reading.png']
messages = [
    "Meditation helps calm the mind.",
    "Exercise boosts your mood.",
    "Deep breathing reduces stress.",
    "Take time to relax and unwind.",
    "Talking to a therapist is healthy.",
    "Support from others strengthens mental health.",
    "Positive affirmations build resilience.",
    "Reading helps you de-stress."
]

# Load and resize images to fit the grid
images = [pygame.image.load(os.path.join(image_folder, file)) for file in image_files]
images = [pygame.transform.scale(img, (CARD_SIZE, CARD_SIZE)) for img in images]  # Resize each image

# Create the board (each image appears twice for matching pairs)
cards = list(range(8)) * 2
random.shuffle(cards)

# Track the state of the cards (hidden or revealed)
revealed = [[False for _ in range(COLS)] for _ in range(ROWS)]
matched = [[False for _ in range(COLS)] for _ in range(ROWS)]

# Q-Learning Parameters
alpha = 0.1  # Learning rate
gamma = 0.95  # Discount factor
epsilon = 1.0  # Exploration rate
epsilon_min = 0.01  # Minimum exploration rate
epsilon_decay = 0.995  # Decay rate for exploration
q_table = np.zeros((ROWS * COLS, ROWS * COLS))  # Q-Table for state-action pairs

# Helper functions
def draw_board():
    screen.fill(WHITE)
    for row in range(ROWS):
        for col in range(COLS):
            card = cards[row * COLS + col]
            rect = pygame.Rect(col * CARD_SIZE, row * CARD_SIZE, CARD_SIZE, CARD_SIZE)
            pygame.draw.rect(screen, BLACK, rect, 2)

            if revealed[row][col] or matched[row][col]:
                # Display the corresponding image
                screen.blit(images[card], (col * CARD_SIZE, row * CARD_SIZE))
            else:
                # Hide the card (draw a red rectangle)
                pygame.draw.rect(screen, RED, rect)
    pygame.display.flip()

def get_card_pos(index):
    row, col = divmod(index, COLS)
    return row, col

def get_card_index(row, col):
    return row * COLS + col

def check_match(card1, card2):
    return cards[card1] == cards[card2]

def ai_action():
    if np.random.rand() < epsilon:
        # Explore: randomly pick a card
        return np.random.choice(range(ROWS * COLS))
    else:
        # Exploit: choose the card with the highest Q-value
        available_cards = [i for i in range(ROWS * COLS) if not matched[i // COLS][i % COLS] and not revealed[i // COLS][i % COLS]]
        return np.argmax(q_table[:, available_cards], axis=1)[-1]

def update_q_table(card1, card2, reward):
    state1, state2 = card1, card2
    best_future_q = np.max(q_table[state1, :])
    q_table[state1, state2] = (1 - alpha) * q_table[state1, state2] + alpha * (reward + gamma * best_future_q)

def show_message(message):
    # Display a message at the bottom of the screen
    text = font.render(message, True, BLACK)
    screen.blit(text, (WIDTH // 10, HEIGHT - 30))
    pygame.display.flip()

# Main game loop
first_card = None
second_card = None
matches_found = 0
running = True
total_moves = 0

# Setting a clock to control the frame rate
clock = pygame.time.Clock()

while running:
    draw_board()

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    if not first_card and not second_card:
        first_card = ai_action()
        row1, col1 = get_card_pos(first_card)
        revealed[row1][col1] = True

    elif first_card and not second_card:
        second_card = ai_action()
        row2, col2 = get_card_pos(second_card)
        revealed[row2][col2] = True
        draw_board()
        pygame.time.wait(500)  # Delay for visual feedback

        # Check if the AI made a match
        if check_match(first_card, second_card):
            matched[row1][col1] = True
            matched[row2][col2] = True
            reward = 1  # Positive reward for a correct match
            matches_found += 1
            show_message(messages[cards[first_card]])
        else:
            revealed[row1][col1] = False
            revealed[row2][col2] = False
            reward = -0.5  # Negative reward for a mismatch

        # Update Q-Table
        update_q_table(first_card, second_card, reward)

        first_card = None
        second_card = None
        total_moves += 1

    # Decay exploration rate
    global epsilon
    if epsilon > epsilon_min:
        epsilon *= epsilon_decay

    # Check for win
    if matches_found == (ROWS * COLS) // 2:
        screen.fill(WHITE)
        text = win_font.render(f"AI Won in {total_moves} moves!", True, BLACK)
        screen.blit(text, (WIDTH // 5, HEIGHT // 2))
        pygame.display.flip()
        pygame.time.wait(2000)
        running = False

    # Control frame rate
    clock.tick(30)

pygame.quit()
```

### Output:

![Screenshot 2024-10-04 131827](https://github.com/user-attachments/assets/299b367d-62c8-4165-81de-bcb63d2bf76a)

![Screenshot 2024-10-04 134753](https://github.com/user-attachments/assets/ebe0115b-e3cc-4bcd-a8d1-4f59d967995f)

### Result:
Thus the Mental Health Awareness Memory Game was implemented using Pygame and Q-Learning.
