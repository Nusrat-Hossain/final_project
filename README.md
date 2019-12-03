# final_project
# import the python packages: pygame and random
import pygame, random

# the width and height of the screen
WIDTH = 1250
HEIGHT = 720

# initialize pygame
pygame.init()

# create the screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))

# load sprites
background = pygame.image.load("background 3.png")
playerImg = pygame.image.load("player 2.png")
asteroidImg = pygame.image.load("asteroid.png")

# the width and height of the player and asteroid, respectively
PLAYER_WIDTH, PLAYER_HEIGHT = 120, 200
ASTEROID_WIDTH, ASTEROID_HEIGHT = 100, 100

# starting x and y coordinates of the player, respectively
initialPlayerX = round(WIDTH//2)
initialPlayerY = HEIGHT - PLAYER_HEIGHT

# in milliseconds
asteroidFreq = 1000

# the number of pixels the player, bullet, or asteroid will move
playerSpeed = 15
bulletSpeed = 20
asteroidSpeed = 5

# radius of the bullet in pixels
bulletRadius = 5

# this function takes in a number (asteroidFreqNumber, which is picked randomly from 0-99) and checks if the number is
# between 40-50 (it has a 1 in 10 chance of being this number), if it is, then it sets the x-coordinate of the asteroid
# to a random integer anywhere between 0 to the width of the screen minus the asteroid width (so that it is not off the
# screen) and then it creates a new asteroid at the random X position and a y position that is equal to
# 0 - asteroid's height so that it appears above the screen, this gets saved to asteroid and then it returns asteroid
# if the number is not between 40-50, the function does not generate a new asteroid and returns false
def generateAsteroid(checkNumber):
    if 40 <= checkNumber <= 50:
        asteroidX = random.randint(0, WIDTH - ASTEROID_WIDTH)
        asteroid = Asteroid(asteroidX, -ASTEROID_HEIGHT)
        return asteroid
    return False


class Player:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def draw(self):
        """
        Draws player onto screen at its x and y co-ordinate
        """
        screen.blit(playerImg, (self.x, self.y))


class Bullet:
    def __init__(self, x, y, radius=bulletRadius):
        self.x = x
        self.y = y
        self.radius = radius
    
    def draw(self):
        pygame.draw.circle(screen, (255, 255, 255), (self.x, self.y), self.radius)

    def hit(self, asteroidX, asteroidY):
        """
       checkX: checks if the x-coordinate of the bullet is within the width of the asteroid (x-coordinate of asteroid to
       x-coordinate of asteroid plus asteroid_width)

       checkY: checks if the y-coordinate of the bullet is within the height of the asteroid (y-coordinate of asteroid to
       y-coordinate of asteroid plus asteroid_height)
        """
        checkX =  asteroidX <= self.x <= asteroidX + ASTEROID_WIDTH
        checkY = asteroidY <= self.y <= asteroidY + ASTEROID_HEIGHT
        return checkX and checkY


class Asteroid:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def draw(self):
        screen.blit(asteroidImg, (self.x, self.y))
    
    def hit(self, playerX, playerY):
        """
       checkX: checks if the x-coordinate of the player is within the width of the asteroid (x-coordinate of asteroid to
       x-coordinate of asteroid plus asteroid_width)

       checkY: checks if the y-coordinate of the player is within the height of the asteroid (y-coordinate of asteroid
       plus asteroid_height)
        """
        checkX =  self.x <= playerX <= self.x + ASTEROID_WIDTH
        checkY = playerY <= self.y + ASTEROID_HEIGHT
        return checkX and checkY


class Score:
    def __init__(self, score):
        self.score = score
        self.score_font = pygame.font.Font('freesansbold.ttf', 32)

    def draw(self):
        score_string = self.score_font.render(str(self.score), True, (0,255,0), (0,0,255)) 
        scoreRect = score_string.get_rect()  
        scoreRect.center = (WIDTH - 100, 50) 
        screen.blit(score_string, scoreRect)


# declare our player at our initial x and y value
player = Player(initialPlayerX, initialPlayerY)

# lists to hold all bullets and all asteroids, respectively
bullets = []
asteroids = []

# set initial score to 0
score = Score(0)

# timers for delays, pygame functions to get time
asteroidTime = pygame.time.get_ticks()
bulletTime = pygame.time.get_ticks()


# main game loop
running = True
while running:
    # get the keys that are pressed
    keys = pygame.key.get_pressed()

    # fill screen and draw background
    screen.fill((180,0,0)) 
    screen.blit(background, (0,0))

    # picks a random number from 0-99, this number will be put into the generateAsteroid function
    asteroidFreqNumber = random.randint(0, 100)
    
    # boundaries for player, player can not leave screen
    if keys[pygame.K_LEFT] and player.x > 0:
        player.x -= playerSpeed
    if keys[pygame.K_RIGHT] and player.x < WIDTH - PLAYER_WIDTH:
        player.x += playerSpeed

    # shooting, spacebar creates a new bullet and adds it to the list 'bullets' if the length of 'bullets' is less than
    # or equal to 5 and if the time between the bullets is greater than 200 milliseconds, otherwise there are no new
    # bullets
    if keys[pygame.K_SPACE] and len(bullets) <= 5:
        currBulletTime = pygame.time.get_ticks()
        if (currBulletTime - bulletTime > 200):
            bulletTime = currBulletTime
            bullets.append(Bullet(player.x + round(PLAYER_WIDTH // 2), player.y, bulletRadius))

    # game over variables
    game_over_font = pygame.font.Font('freesansbold.ttf', 64)
    game_over_string = game_over_font.render("GAME OVER", True, (255, 0, 0), (0, 0, 0))
    gameOverRect = game_over_string.get_rect()
    gameOverRect.center = (WIDTH//2, HEIGHT//2)

    # asteroids
    for asteroid in asteroids:
        # make asteroids fall down
        asteroid.y += asteroidSpeed
        # if asteroid hits player, that asteroid gets popped out and removed from the list 'asteroids', screen says GAME
        # OVER and program stops because running equals false
        if asteroid.hit(player.x, player.y):
            asteroids.pop(asteroids.index(asteroid))
            screen.blit(game_over_string, gameOverRect)
            pygame.time.delay(500)
            running = False
        # if asteroid leaves the screen, it gets popped out and removed from the list 'asteroids', if it does not, then
        # no new asteroids will be generated after the 9th asteroid because they are only generated if the length of
        # the list is less than 10
        if asteroid.y > HEIGHT:
            asteroids.pop(asteroids.index(asteroid))
    
    for bullet in bullets:
        # makes bullet travel up screen
        bullet.y -= bulletSpeed
        for asteroid in asteroids:
            # if bullet hits asteroid, bullet and asteroid get removed from their lists and the score increases by 1
            if bullet.hit(asteroid.x, asteroid.y):
                bullets.pop(bullets.index(bullet))
                asteroids.pop(asteroids.index(asteroid))
                score.score += 1
        # if the bullet leaves the screen, it gets removed from the list, if it does not, then no new bullets will be
        # generated after the 5th bullet because they are only generated if the length of the list is less than or
        # equal to 5
        if bullet.y < 0:
            bullets.pop(bullets.index(bullet))

    # if the x button on the window is pressed, running equals false and the program stops
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # call draw method to draw player
    player.draw()

    # adds a new asteroid to the list 'asteroids' if the length of 'asteroids' is less than 10, the generateAsteroid
    # function does not return false, and the time between asteroids is greater than 1000 milliseconds (asteroidFreq)
    if len(asteroids) < 10:
        currAsteroidTime = pygame.time.get_ticks()
        newAsteroid = generateAsteroid(asteroidFreqNumber)
        if newAsteroid and currAsteroidTime - asteroidTime > asteroidFreq:
            asteroidTime = currAsteroidTime
            asteroids.append(newAsteroid)

    # calls the draw method for bullet
    for bullet in bullets:
        bullet.draw()

    # calls the draw method for asteroid
    for asteroid in asteroids:
        asteroid.draw()

    # calls the draw method for score
    score.draw()
    # updates the screen
    pygame.display.update()
