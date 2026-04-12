def reproduce(population, target):
    mating_pool = []
    for rocket in population:
        # Умножаем фитнес на большое число, чтобы создать список "претендентов"
        n = int(rocket.fitness * 10000)
        mating_pool.extend([rocket] * n)

    # Если никто не выжил, берем всех, чтобы не было ошибки
    if not mating_pool:
        mating_pool = population

    new_population = []
    # Цикл создания новых 100 ракет
    for _ in range(POPULATION_SIZE):
        # 1. Выбираем двух случайных родителей из пула успешных
        parent_a = random.choice(mating_pool).brain
        parent_b = random.choice(mating_pool).brain
        
        # 2. Создаем "мозг" для ребенка
        child_brain = Brain(LIFESPAN)
        
        # 3. Кроссовер: берем часть генов от одного родителя, часть от другого
        mid = random.randint(0, LIFESPAN)
        child_brain.genes = parent_a.genes[:mid] + parent_b.genes[mid:]
        
        # 4. Мутация: добавляем случайности
        child_brain.mutate()
        
        # 5. Добавляем новую ракету в новое поколение
        new_population.append(Rocket(child_brain))
    
    return new_population
