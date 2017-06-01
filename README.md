# Density
import pygame, sys, random, math, os

# Runs imported module
pygame.init()
pygame.mixer.init()

# Constants
WIN_W = 1200
WIN_H = 670

WHITE = (255, 255, 255)
BLACK = (0,0,0)
YELLOW = (255, 255, 0)
RED = (255, 0, 0)
BLUE = (0, 0, 255)
PINK = (255, 20, 147)
CYAN = (0, 255, 255)
LIME = (50, 205, 50)
PURPLE = (160, 32, 240)

SHIP_WIDTH = SHIP_HEIGHT = WIN_W / 120

PILL_WIDTH = 7
PILL_HEIGHT = 250

screen = pygame.display.set_mode((WIN_W, WIN_H), pygame.SRCALPHA)


# Classes
class Ship(pygame.sprite.Sprite):
 def __init__(self, x, y, player):
     pygame.sprite.Sprite.__init__(self)
     self.player = player
     self.speed = 5
     self.image = pygame.Surface((SHIP_WIDTH, SHIP_HEIGHT)).convert()
     self.rect = self.image.get_rect()
     self.rect = self.rect.move(x, y)
     self.density = 169

 def grow(self):
     self.image = pygame.transform.scale(self.image,(int(math.sqrt(int(self.density))), int(math.sqrt(int(self.density)))))
     self.rect = pygame.Rect(self.rect.x, self.rect.y, int(math.sqrt(int(self.density))), int(math.sqrt(int(self.density))))

 def update(self, pill_group, ship_left, ship_right):
     key = pygame.key.get_pressed()

     # Ship Movement
     if self.player == "left":
         if key[pygame.K_w]:
             self.rect.y -= self.speed
         if key[pygame.K_s]:
             self.rect.y += self.speed
         if key[pygame.K_a]:
             self.rect.x -= self.speed
         if key[pygame.K_d]:
             self.rect.x += self.speed

     if self.player == "right":
         if key[pygame.K_UP]:
             self.rect.y -= self.speed
         if key[pygame.K_DOWN]:
             self.rect.y += self.speed
         if key[pygame.K_LEFT]:
             self.rect.x -= self.speed
         if key[pygame.K_RIGHT]:
             self.rect.x += self.speed

     # Keep Ship Movement Inbounds
     if self.rect.y < WIN_H / 15:
         self.rect.y = WIN_H / 15
     if self.rect.y > WIN_H - int(math.sqrt(int(self.density))):
         self.rect.y = WIN_H - int(math.sqrt(int(self.density)))
     if self.player == "left":
         if self.rect.x > WIN_W / 2 - int(math.sqrt(int(self.density))):
             self.rect.x = WIN_W / 2 - int(math.sqrt(int(self.density)))
         if self.rect.x < 0:
             self.rect.x = 0
     if self.player == "right":
         if self.rect.x < WIN_W / 2:
             self.rect.x = WIN_W / 2
         if self.rect.x > WIN_W - int(math.sqrt(int(self.density))):
             self.rect.x = WIN_W - int(math.sqrt(int(self.density)))

     collisions = pygame.sprite.spritecollide(self, pill_group, True)
     for key in collisions:
         self.density += (key.density * 50)

     self.grow()


class Pill(pygame.sprite.Sprite):
 def __init__(self, xval, density):
     pygame.sprite.Sprite.__init__(self)
     self.x = xval
     self.y = WIN_H / 15
     self.speed = 5
     self.density = density
     self.image = pygame.Surface((PILL_WIDTH, PILL_HEIGHT)).convert()
     self.image.fill(self.set_color(density))
     self.rect = self.image.get_rect()
     self.rect = self.rect.move(self.x, self.y)

 def update(self):
     self.y += self.speed
     self.rect = self.image.get_rect()
     self.rect = self.rect.move(self.x, self.y)
     if self.rect.y > WIN_H:
         self.kill()

 def set_color(self, density):
     if density == 1:
         return YELLOW
     if density == 2:
         return RED
     if density == 3:
         return BLUE
     if density == 4:
         return BLACK
     if density == 5:
         self.density = -1
         return PURPLE
     if density == 6:
         self.density = 10
         return LIME

def gen_random():
 xval_density = []
 for _ in range(3000):
     xval_density.append((
         random.randrange(0, (WIN_W / 2) - PILL_WIDTH), int(random.choice('1111111111111111111111111111111111111111'
                                                                          '22222222222222222233334443'
                                                                          '3333334444444444555555555'
                                                                          '555555555555555555556'))))
 return xval_density


class Text(pygame.sprite.Sprite):
 def __init__(self, player, size, color, position, font):
     pygame.sprite.Sprite.__init__(self)
     self.player = player
     self.color = color
     self.position = position
     self.font = pygame.font.Font(font, size)

 def update(self, left_ship_density, right_ship_density):
     if self.player == "left":
         text = "mass: " + str(left_ship_density - 169)
     else:
         text = "mass: " + str(right_ship_density - 169)

     self.image = self.font.render(str(text), 1, self.color)
     self.rect = self.image.get_rect()
     self.rect.move_ip(self.position[0] - self.rect.width / 2, self.position[1])

def main():
 # Initialize variables
 TIMER = 0
 intro = play = outro = True

 fps = 60
 clock = pygame.time.Clock()

 xval_density = gen_random()
 max_pill_count = 2 * len(xval_density)
 pill_count = 0

 # Create Game Objects
 ship_left = Ship((WIN_W / 4) - (SHIP_WIDTH / 2), WIN_H - (SHIP_HEIGHT * 4), 'left')
 ship_right = Ship(((WIN_W * 3) / 4) - (SHIP_WIDTH / 2), WIN_H - (SHIP_HEIGHT * 4), 'right')

 vert_partition = pygame.Surface((1, WIN_H))

 left_score = Text("left", 40, BLACK, [(WIN_W / 5), 20], None)
 right_score = Text("right", 40, BLACK, [(WIN_W / 1.25), 20], None)

 # Create Groups
 ship_group = pygame.sprite.Group()
 ship_group.add(ship_left)
 ship_group.add(ship_right)

 pill_group = pygame.sprite.Group()

 beg_time = pygame.time.get_ticks()

 score_group = pygame.sprite.Group()
 score_group.add(left_score, right_score)

 # Colors
 colors = [YELLOW, PINK, CYAN, RED, LIME, PURPLE, BLUE]

 # Intro Loop
 while intro:
     cur_time = pygame.time.get_ticks()

     if ((cur_time - beg_time) % 250) < 17:
         t_color = colors[random.randrange(0,6,1)]
         c_color = colors[random.randrange(0,6,1)]

     big_font = pygame.font.Font(None, 250)
     small_font = pygame.font.Font(None, 100)
     title = big_font.render("DENSITY", 1, t_color)
     click = small_font.render("---CLICK TO PLAY---", 1, c_color)

     title_rect = title.get_rect()
     title_rect.centerx = screen.get_rect().centerx
     click_rect = (WIN_W / 2 - 300, WIN_H / 2)
     #click_rect.centerx = screen.get_rect().centerx

     screen.fill(WHITE)
     screen.blit(title, title_rect)

     # Blinking Text: Click here to start
     if ((cur_time - beg_time) % 1000) < 500:
         screen.blit(click, click_rect)

     # Checks if window exit button pressed
     for event in pygame.event.get():
         if event.type == pygame.QUIT:
             sys.exit()
         elif event.type == pygame.MOUSEBUTTONDOWN or pygame.key.get_pressed()[pygame.K_RETURN] != 0:
             screen.blit(title, title_rect)
             screen.blit(click, click_rect)
             pygame.display.flip()
             pygame.time.wait(1500)
             intro = False

     # Writes to main surface
     pygame.display.flip()

 # Game Loop
 while play:
     # Checks if window exit button pressed
     for event in pygame.event.get():
         if event.type == pygame.QUIT:
             sys.exit()

         # Keypresses
         elif event.type == pygame.KEYDOWN:
             if event.key == pygame.K_ESCAPE:
                 pygame.quit()
                 sys.exit()

     # Adding Pills
     if pill_count < max_pill_count and TIMER % 1 == 0:
         pill = Pill(xval_density[pill_count][0], xval_density[pill_count][1])
         pill2 = Pill((WIN_W / 2) + xval_density[pill_count][0], xval_density[pill_count][1])
         pill_group.add(pill)
         pill_group.add(pill2)
         pill_count += 1

     # Update Groups
     ship_group.update(pill_group, ship_left.density, ship_right.density)
     pill_group.update()

     if ship_left.density > 10000:
         if ship_left.density < 1500000:
             ship_left.density += 5000
         if ship_right.density > 169:
             ship_right.density -= 250

     elif ship_right.density > 10000:
         if ship_right.density < 1500000:
             ship_right.density += 5000
         if ship_left.density > 169:
             ship_left.density -= 250

     if ship_left.density > 1499999 and ship_right.density == 169:
         play = False
         outro = True

     elif ship_right.density > 1499999 and ship_left.density == 169:
         play = False
         outro = True

     if ship_right.density < 169:
         ship_right.density = 169

     if ship_left.density < 169:
         ship_left.density = 169

     score_group.update(ship_left.density, ship_right.density)

     # Print Groups
     screen.fill(WHITE)
     ship_group.draw(screen)
     pill_group.draw(screen)
     screen.blit(vert_partition, (WIN_W / 2, WIN_H / 15))
     score_group.draw(screen)

     TIMER += 1
     # Limits frames per iteration of while loop
     clock.tick(fps)
     # Writes to main surface
     pygame.display.flip()

 # Outro
 while outro:
     cur_time = pygame.time.get_ticks()

     if ((cur_time - beg_time) % 250) < 17:
         t_color = colors[random.randrange(0, 6, 1)]
         c_color = colors[random.randrange(0, 6, 1)]

     big_font = pygame.font.Font(None, 150)
     small_font = pygame.font.Font(None, 75)

     if ship_left.density < ship_right.density:
         winner = big_font.render("RIGHT PLAYER WINS!!!", 1, t_color)
         click = small_font.render("---CLICK TO PLAY AGAIN---", 1, c_color)

     if ship_right.density < ship_left.density:
         winner = big_font.render("LEFT PLAYER WINS!!!", 1, t_color)
         click = small_font.render("---CLICK TO PLAY AGAIN---", 1, c_color)

     winner_rect = winner.get_rect()
     winner_rect.centerx = screen.get_rect().centerx
     click_rect = (WIN_W / 2 - 300, WIN_H / 2)
     # click_rect.centerx = screen.get_rect().centerx

     screen.fill(WHITE)
     screen.blit(winner, winner_rect)

     # Blinking Text: Click here to start
     if ((cur_time - beg_time) % 1000) < 500:
         screen.blit(click, click_rect)

     # Checks if window exit button pressed
     for event in pygame.event.get():
         if event.type == pygame.QUIT:
             sys.exit()
         elif event.type == pygame.MOUSEBUTTONDOWN or pygame.key.get_pressed()[pygame.K_RETURN] != 0:
             screen.blit(winner, winner_rect)
             screen.blit(click, click_rect)
             pygame.display.flip()
             pygame.time.wait(1500)
             play = True
             outro = False
             main()

     # Writes to main surface
     pygame.display.flip()


if __name__ == "__main__":
 # Force static position of screen
 os.environ['SDL_VIDEO_CENTERED'] = '1'

 # Runs imported module
 pygame.init()

main()




