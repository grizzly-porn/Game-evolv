import pygame
import random
import math

# Настройки экрана
WIDTH, HEIGHT = 800, 500
FPS = 60

# Цвета
WHITE = (255, 255, 255)
BLACK = (20, 20, 30)
NEON_BLUE = (0, 150, 255)
NEON_RED = (255, 50, 100)

class SimpleNeuralNet:
    """ Упрощенная нейросеть с 'зашитыми' весами для игры """
    def __init__(self):
        # Веса подобраны так, чтобы ИИ был очень точным
        self.w1 = 0.8  # Вес для позиции мяча
        self.w2 = -0.8 # Вес для позиции собственной ракетки
        self.bias = 0  # Смещение

    def predict(self, ball_y, paddle_y):
        # Математика нейрона: сумма входов * веса
        # Если результат положительный - идем вниз, если отрицательный - вверх
        output = (ball_y * self.w1) + (paddle_y * self.w2) + self.bias
        
        if output > 5:
            return 1  # Вниз
        elif output < -5:
            return -1 # Вверх
        return 0 # Стоять

class Ball:
    def __init__(self):
        self.reset()

    def reset(self):
        self.x = WIDTH // 2
        self.y = HEIGHT // 2
        self.speed_x = 7 * random.choice([-1, 1])
        self.speed_y = random.uniform(-5, 5)

    def move(self):
        self.x += self.speed_x
        self.y += self.speed_y

        if self.y <= 0 or self.y >= HEIGHT:
            self.speed_y *= -1

class Paddle:
    def __init__(self, x, color):
        self.rect = pygame.Rect(x, HEIGHT//2 - 40, 15, 80)
        self.color = color
        self.speed = 6

    def move(self, direction):
        if direction == -1 and self.rect.top > 0:
            self.rect.y -= self.speed
        if direction == 1 and self.rect.bottom < HEIGHT:
            self.rect.y += self.speed

def main():
    pygame.init()
    screen = pygame.display.set_mode((WIDTH, HEIGHT))
    pygame.display.set_caption("Neural Pong: Player vs Pre-trained AI")
    clock = pygame.time.Clock()
    font = pygame.font.SysFont("Arial", 24)

    ball = Ball()
    player = Paddle(20, NEON_BLUE)
    ai_paddle = Paddle(WIDTH - 35, NEON_RED)
    ai_brain = SimpleNeuralNet()

    score_player = 0
    score_ai = 0

    running = True
    while running:
        screen.fill(BLACK)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

        # Управление игрока (мышь)
        mouse_y = pygame.mouse.get_pos()[1]
        player.rect.centery = mouse_y

        # --- ЛОГИКА НЕЙРОСЕТИ ---
        # Передаем ИИ положение мяча и его собственное положение
        decision = ai_brain.predict(ball.y, ai_paddle.rect.centery)
        ai_paddle.move(decision)

        # Движение мяча
        ball.move()

        # Отскоки от ракеток
        if ball.x <= 40 and player.rect.colliderect(ball.x, ball.y, 10, 10):
            ball.speed_x *= -1.1 # Немного ускоряем при каждом отскоке
            ball.speed_y = (ball.y - player.rect.centery) * 0.2
        
        if ball.x >= WIDTH - 50 and ai_paddle.rect.colliderect(ball.x, ball.y, 10, 10):
            ball.speed_x *= -1.1
            ball.speed_y = (ball.y - ai_paddle.rect.centery) * 0.2

        # Голы
        if ball.x < 0:
            score_ai += 1
            ball.reset()
        elif ball.x > WIDTH:
            score_player += 1
            ball.reset()

        # Отрисовка
        pygame.draw.aaline(screen, WHITE, (WIDTH//2, 0), (WIDTH//2, HEIGHT))
        pygame.draw.ellipse(screen, WHITE, (ball.x-10, ball.y-10, 20, 20))
        pygame.draw.rect(screen, player.color, player.rect)
        pygame.draw.rect(screen, ai_paddle.color, ai_paddle.rect)

        # Текст
        p_text = font.render(f"Игрок: {score_player}", True, NEON_BLUE)
        ai_text = font.render(f"Нейросеть: {score_ai}", True, NEON_RED)
        screen.blit(p_text, (WIDTH//4, 20))
        screen.blit(ai_text, (WIDTH*0.6, 20))
        
        # Визуализация "мыслей" ИИ
        think_text = font.render("ИИ: анализирую Y...", True, (100, 100, 100))
        screen.blit(think_text, (WIDTH - 200, HEIGHT - 30))

        pygame.display.flip()
        clock.tick(FPS)

    pygame.quit()

if __name__ == "__main__":
    main()
    
