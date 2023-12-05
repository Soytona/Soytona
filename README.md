import pygame
import sys
import random

# Configuración del juego
pygame.init()
width, height = 300, 600
block_size = 30
screen = pygame.display.set_mode((width, height))
clock = pygame.time.Clock()

# Colores
black = (0, 0, 0)
white = (255, 255, 255)
colors = [(0, 255, 0), (0, 0, 255), (255, 0, 0), (255, 255, 0), (255, 0, 255)]

# Piezas de Tetris
tetris_shapes = [
    [[1, 1, 1],
     [0, 1, 0]],

    [[0, 1, 1],
     [1, 1, 0]],

    [[1, 1, 0],
     [0, 1, 1]],

    [[1, 1, 1, 1]],

    [[1, 1],
     [1, 1]],

    [[1, 1, 1],
     [1, 0, 0]],
    
    [[1, 1, 1],
     [0, 0, 1]]
]

# Clase para las piezas
class Piece:
    def __init__(self, shape, color):
        self.shape = shape
        self.color = color
        self.x = width // 2 - len(shape[0]) // 2
        self.y = 0

# Función para dibujar una pieza
def draw_piece(piece):
    for i in range(len(piece.shape)):
        for j in range(len(piece.shape[i])):
            if piece.shape[i][j] != 0:
                pygame.draw.rect(screen, piece.color, (piece.x * block_size + j * block_size,
                                                       piece.y * block_size + i * block_size,
                                                       block_size, block_size))

# Función principal del juego
def tetris_game():
    score = 0
    grid = [[0] * (width // block_size) for _ in range(height // block_size)]

    current_piece = Piece(random.choice(tetris_shapes), random.choice(colors))
    next_piece = Piece(random.choice(tetris_shapes), random.choice(colors))

    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT and is_valid_move(current_piece, grid, dx=-1):
                    current_piece.x -= 1
                elif event.key == pygame.K_RIGHT and is_valid_move(current_piece, grid, dx=1):
                    current_piece.x += 1
                elif event.key == pygame.K_DOWN and is_valid_move(current_piece, grid, dy=1):
                    current_piece.y += 1
                elif event.key == pygame.K_UP:
                    rotated_piece = rotate_piece(current_piece)
                    if is_valid_move(rotated_piece, grid):
                        current_piece.shape = rotated_piece.shape

        if is_valid_move(current_piece, grid, dy=1):
            current_piece.y += 1
        else:
            merge_piece(current_piece, grid)
            clear_lines(grid)
            current_piece = next_piece
            next_piece = Piece(random.choice(tetris_shapes), random.choice(colors))

            if not is_valid_move(current_piece, grid):
                game_over()

        screen.fill(black)
        draw_grid(grid)
        draw_piece(current_piece)
        draw_next_piece(next_piece)

        pygame.display.flip()
        clock.tick(5)

# Función para comprobar si un movimiento es válido
def is_valid_move(piece, grid, dx=0, dy=0):
    for i in range(len(piece.shape)):
        for j in range(len(piece.shape[i])):
            if piece.shape[i][j] != 0:
                x = piece.x + j + dx
                y = piece.y + i + dy
                if x < 0 or x >= len(grid[0]) or y >= len(grid) or (y >= 0 and grid[y][x] != 0):
                    return False
    return True

# Función para rotar una pieza
def rotate_piece(piece):
    rotated_shape = list(zip(*reversed(piece.shape)))
    return Piece(rotated_shape, piece.color)

# Función para fusionar una pieza en el tablero
def merge_piece(piece, grid):
    for i in range(len(piece.shape)):
        for j in range(len(piece.shape[i])):
            if piece.shape[i][j] != 0:
                grid[piece.y + i][piece.x + j] = piece.color

# Función para limpiar líneas completas en el tablero
def clear_lines(grid):
    lines_to_clear = [i for i, row in enumerate(grid) if all(row)]
    for line in lines_to_clear:
        del grid[line]
        grid.insert(0, [0] * len(grid[0]))

# Función para dibujar el tablero
def draw_grid(grid):
    for i in range(len(grid)):
        for j in range(len(grid[i])):
            if grid[i][j] != 0:
                pygame.draw.rect(screen, grid[i][j], (j * block_size, i * block_size, block_size, block_size), 0)

# Función para dibujar la próxima pieza
def draw_next_piece(piece):
    for i in range(len(piece.shape)):
        for j in range(len(piece.shape[i])):
            if piece.shape[i][j] != 0:
                pygame.draw.rect(screen, piece.color, (width + j * block_size,
                                                       i * block_size,
                                                       block_size, block_size))

# Función para manejar el final del juego
def game_over():
    pygame.quit()
    sys.exit()

# Inicia el juego
tetris_game()
