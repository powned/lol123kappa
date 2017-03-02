# Shump Game
import pygame
import random
from os import path
img_dir = path.join(path.dirname(__file__),"img")
snd_dir = path.join(path.dirname(__file__),"snd")

fps = 60
width = 480
height = 600
#colors
white = (255 , 255 , 255)
black = (0,0,0)
red = (255, 0 , 0)
green = (0,255, 0)
blue = (0,0 , 255)
yellow = (195,224,2)
#initialize pygame and create windows 
pygame.init()
pygame.mixer.init()
screen = pygame.display.set_mode((width , height))
pygame.display.set_caption("The Guardien")
clock = pygame.time.Clock()

#draw text
font_name = pygame.font.match_font("arial")
def draw_text(surf, text, size, x, y):
 font = pygame.font.Font(font_name, size)
 text_surface = font.render(text, True , white)
 text_rect = text_surface.get_rect()
 text_rect.midtop = (x, y)
 surf.blit(text_surface , text_rect)
#Mobs , player , ship
class Ship(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(ship_img, (50,38))
        self.image.set_colorkey(black)
        self.rect = self.image.get_rect()
        self.radius = 21
        self.rect.centerx = width/2
        self.rect.bottom = height -10
        self.speedx =0
    def update(self):
        self.speedx =0
        keystate = pygame.key.get_pressed()
        if keystate[pygame.K_LEFT] or keystate[pygame.K_a]:
            self.speedx = -5
        if keystate[pygame.K_RIGHT] or keystate[pygame.K_d]:
            self.speedx = 5
        self.rect.x += self.speedx
        if self.rect.right > width:
            self.rect.right = width
        if self.rect.left < 0 :
            self.rect.left = 0
    def shoot(self):
     Bullet = bullet(self.rect.centerx , self.rect.top)
     all_sprites.add(Bullet)
     bullets.add(Bullet)
     shot_snd.play()


class Mob(pygame.sprite.Sprite):
 def __init__(self):
  pygame.sprite.Sprite.__init__(self)
  self.image_orig = random.choice(mob_images)
  self.image_orig.set_colorkey(black)
  self.image = self.image_orig.copy()
  self.rect = self.image.get_rect()
  self.radius = int(self.rect.width *0.9/2)
  self.rect.x = random.randrange(width - self.rect.width)
  self.rect.y = random.randrange(-150,-100)
  self.speedy = random.randrange(1,8)
  self.speedx = random.randrange(-3,3)
  self.rot = 0
  self.rot_speed = random.randrange(-8, 8)
  self.last_update = pygame.time.get_ticks()
 def rotate(self):
     now = pygame.time.get_ticks()
     if now - self.last_update > 50:
         self.last_update = now
         self.rot = (self.rot + self.rot_speed) % 360
         new_image = pygame.transform.rotate(self.image_orig , self.rot)
         old_center = self.rect.center
         self.image = new_image
         self.rect = self.image.get_rect()
         self.rect.center = old_center
 def update(self):
     self.rotate()
     self.rect.x += self.speedx
     self.rect.y += self.speedy
     if self.rect.top > height +10 or self.rect.left < -25 or self.rect.right > width +20:
         self.rect.x = random.randrange(width +10)
         self.rect.y = random.randrange(-100,-40)
         self.speedy = random.randrange(2, 8)

     
class bullet(pygame.sprite.Sprite):
    def __init__(self , x , y):
        pygame.sprite.Sprite.__init__(self)
        self.image = bullet_img
        self.image.set_colorkey(black)
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x 
        self.speedy = -10 
                     
    def update(self):
        self.rect.y += self.speedy
        #kill it if it moves off the top of teh screen 
        if self.rect.bottom < 0:
            self.kill() 
#load game grafics
background = pygame.image.load(path.join(img_dir, "Bg.png")).convert()
background_rect = background.get_rect()
ship_img = pygame.image.load(path.join(img_dir,"playerShip1_red.png" )).convert()
bullet_img = pygame.image.load(path.join(img_dir,"laserBlue12.png" )).convert()
mob_images =[]
mob_list = ["meteorBrown_big3.png", "meteorGrey_big4.png", "meteorBrown_med1.png",
            "meteorBrown_med3.png", "meteorGrey_med1.png", "meteorGrey_med2.png",
            "meteorGrey_tiny1.png"]
for img in mob_list:
    mob_images.append(pygame.image.load(path.join(img_dir, img)).convert())
#load images
shot_snd = pygame.mixer.Sound(path.join(snd_dir, "lg_converted.wav"))
expl_sounds = []
for snd in ["Explosion.wav"]:
 expl_sounds.append(pygame.mixer.Sound(path.join(snd_dir , snd)))


#Sprits (Profiles)
all_sprites = pygame.sprite.Group()
mobs = pygame.sprite.Group()
bullets = pygame.sprite.Group()
player = Ship()
all_sprites.add(player)
for i in range(8):
    m = Mob()
    all_sprites.add(m)
    mobs.add(m)
score = 0
#game loop
running = True
while running:
    clock.tick(fps)
    #procces input (events)
    for event in pygame.event.get():
     if event.type == pygame.QUIT:
         running = False
     elif event.type == pygame.KEYDOWN:
        if event.key == pygame.K_SPACE:
            player.shoot()
    #update
    all_sprites.update()
    #cqheck to see if the bullet destroyed the Mob
    hits1 = pygame.sprite.groupcollide(bullets, mobs, True , True)
    for hit in hits1:
        score +=1 + m.radius
        random.choice(expl_sounds).play()
        m = Mob()
        all_sprites.add(m)
        mobs.add(m)
    #check if a mob hit a player
    hits = pygame.sprite.spritecollide(player , mobs, False, pygame.sprite.collide_circle)
    if hits:
        running = False
    
    #draw render
    screen.fill(black)
    screen.blit(background , background_rect)
    all_sprites.draw(screen)
    draw_text(screen, str(score), 18, width/2, 10)
    #after drawing everything
    pygame.display.flip()


pygame.quit()     

