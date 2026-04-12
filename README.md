# Game-evolv
import pygame
import random
import math

# Настройки
WIDTH, HEIGHT = 800, 600
FPS = 60
POPULATION_SIZE = 100
LIFESPAN = 300  # Сколько кадров живет одно поколение
MUTATION_RATE = 0.02 # Шанс мутации (2%)

# Цвета
WHITE = (255, 255, 255)
BLACK = (10, 10, 15)
RED = (255, 50, 50)
GREEN = (0, 255, 100)
BLUE = (50, 150, 255)

class Brain:
    def __init__(self, size):
        # Гены - это список векторов (сил), которые толкают ракету
        self.genes = []
        for _ in range(size):
            angle = random.uniform(0, 2 * math.pi)
            force = [math.cos(angle) * 0.5, math.sin(angle) * 0.5]
            self.genes.append(force)

    def mutate(self):
        for i in range(len(self.genes)):
            if random.random() < MUTATION_RATE:
                angle = random.uniform(0, 2 * math.pi)
                self.genes[i] = [math.cos(angle) * 0.5, math.sin(angle) * 0.5]

class Rocket:
    def __init__(self, brain=None):
        self.pos = [WIDTH // 2, HEIGHT - 20]
        self.vel = [0, 0]
        self.acc = [0, 0]
        self.brain = brain if brain else Brain(LIFESPAN)
        self.fitness = 0
        self.completed = False
        self.crashed = False

    def apply_force(self, force):
        self.acc[0] += force[0]
        self.acc[1] += force[1]

    def update(self, step, target, obstacles):
        if self.completed or self.crashed:
            return

        self.apply_force(self.brain.genes[step])
        self.vel[0] += self.acc[0]
        self.vel[1] += self.acc[1]
        self.pos[0] += self.vel[0]
        self.pos[1] += self.vel[1]
        self.acc = [0, 0]

        # Проверка столкновения с целью
        dist = math.hypot(self.pos[0] - target.x, self.pos[1] - target.y)
        if dist < 20:
            self.completed = True
            self.pos = [target.x, target.y]

        # Проверка столкновения с препятствиями и границами
        for obs in obstacles:
            if obs.collidepoint(self.pos[0], self.pos[1]):
                self.crashed = True
        
        if self.pos[0] < 0 or self.pos[0] > WIDTH or self.pos[1] < 0 or self.pos[1] > HEIGHT:
            self.crashed = True

    def calculate_fitness(self, target):
        dist = math.hypot(self.pos[0] - target.x, self.pos[1] - target.y)
        # Чем ближе к цели, тем выше приспособленность (инверсия расстояния)
        self.fitness = 1 / (dist**2 + 1)
        if self.completed: self.fitness *= 10 # Бонус за финиш
                if self.crashed: self.fitness /= 5    # Штраф за аварию

    def draw(self, screen):
        color = GREEN if self.completed else (RED if self.crashed else WHITE)
        pygame.draw.circle(screen, color, (int(self.pos[0]), int(self.pos[1])), 3)

def reproduce(population, target):
    # Отбор лучших (метод "рулетки")
    mating_pool = []
    for rocket in population:
        n = int(rocket.fitness * 10000) # Чем выше фитнес, тем больше шансов попасть в пул
        mating_pool.extend([rocket] * n)

    new_population = []
    for _ in range(POPULATION_SIZE):
        parent_a = random.choice(mating_pool).brain
        parent_b = random.choice(mating_pool).brain
        
        # Кроссовер (смешивание генов)
        child_brain = Brain(LIFESPAN)
        mid = random.randint(0, LIFESPAN)
        child_brain.genes = parent_a.genes[:mid] + parent_b.genes[mid:]
        
        # Мутация
        child_brain.mutate()
        new_population.append(Rocket(child_brain))
    
    return new_population

def main():
    pygame.init()
    screen = pygame.display.set_mode((WIDTH, HEIGHT))
    clock = pygame.time.Clock()
    font = pygame.font.SysFont("Arial", 18)

    target = pygame.Rect(WIDTH // 2 - 10, 50, 20, 20)
    obstacles = [
        pygame.Rect(200, 300, 400, 20), # Длинная преграда посередине
    ]

    population = [Rocket() for _ in range(POPULATION_SIZE)]
    generation = 1
    step = 0

    running = True
    while running:
        screen.fill(BLACK)
        
        for event in pygame.event.get():
            if event.type == pygame.QUIT: running = False

        # Отрисовка цели и препятствий
        pygame.draw.ellipse(screen, GREEN, target)
        for obs in obstacles:
            pygame.draw.rect(screen, BLUE, obs)

        # Жизненный цикл поколения
        for rocket in population:
            rocket.update(step, target, obstacles)
            rocket.draw(screen)

        step += 1
        
        # Если время вышло - создаем новое поколение
        if step >= LIFESPAN:
            for rocket in population:
                rocket.calculate_fitness(target)
            population = reproduce(population, target)
            step = 0
            generation += 1

        # Текст
        info = font.render(f"Поколение: {generation} | Шаг: {step}/{LIFESPAN}", True, WHITE)
        screen.blit(info, (10, 10))

        pygame.display.flip()
        clock.tick(FPS)

    pygame.quit()

if __name__ == "__main__":
    main()
