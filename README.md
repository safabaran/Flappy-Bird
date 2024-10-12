import pygame
import random

# Oyun ayarları
WIDTH, HEIGHT = 400, 600
FPS = 60
GRAVITY = 0.5
JUMP_STRENGTH = -10

# Renkler
WHITE = (255, 255, 255)
GREEN = (0, 255, 0)
RED = (255, 0, 0)

# Pygame'i başlat
pygame.init()
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Flappy Bird")
clock = pygame.time.Clock()
font = pygame.font.SysFont('Arial', 40)

# Kuş sınıfı
class Bird:
    def __init__(self):
        self.x = 50
        self.y = HEIGHT // 2
        self.velocity = 0
        self.image = pygame.Surface((30, 30))
        self.image.fill(WHITE)

    def jump(self):
        self.velocity = JUMP_STRENGTH

    def update(self):
        self.velocity += GRAVITY
        self.y += self.velocity

    def draw(self, surface):
        surface.blit(self.image, (self.x, self.y))

# Boru sınıfı
class Pipe:
    def __init__(self):
        self.x = WIDTH
        self.height = random.randint(150, 450)
        self.gap = 150
        self.passed = False
        self.top_rect = pygame.Rect(self.x, 0, 50, self.height)
        self.bottom_rect = pygame.Rect(self.x, self.height + self.gap, 50, HEIGHT - self.height - self.gap)

    def update(self):
        self.x -= 5
        self.top_rect.x = self.x
        self.bottom_rect.x = self.x

    def draw(self, surface):
        pygame.draw.rect(surface, GREEN, self.top_rect)
        pygame.draw.rect(surface, GREEN, self.bottom_rect)

    def off_screen(self):
        return self.x < -50

# Oyun döngüsü
def main():
    bird = Bird()
    pipes = [Pipe()]
    score = 0
    run = True

    while True:
        clock.tick(FPS)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                return
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    if not run:
                        main()  # Yeniden başlat
                    else:
                        bird.jump()

        if run:
            bird.update()

            # Kuşun ekran sınırlarını kontrol et
            if bird.y >= HEIGHT or bird.y <= 0:
                run = False

            for pipe in pipes:
                pipe.update()
                if pipe.off_screen():
                    pipes.remove(pipe)
                    score += 1
                if pipe.x < bird.x and not pipe.passed:
                    pipe.passed = True
                    pipes.append(Pipe())

                # Kuş ve boru çarpışması kontrolü
                if pipe.top_rect.colliderect(pygame.Rect(bird.x, bird.y, 30, 30)) or \
                   pipe.bottom_rect.colliderect(pygame.Rect(bird.x, bird.y, 30, 30)):
                    run = False

            screen.fill((0, 0, 0))
            bird.draw(screen)
            for pipe in pipes:
                pipe.draw(screen)
        else:
            # Oyun sona erdiğinde mesaj göster
            game_over_text = font.render("Game Over!", True, RED)
            restart_text = font.render("Press SPACE to Restart", True, WHITE)
            screen.blit(game_over_text, (WIDTH // 4, HEIGHT // 3))
            screen.blit(restart_text, (WIDTH // 8, HEIGHT // 2))

        pygame.display.flip()

if __name__ == "__main__":
    main()
