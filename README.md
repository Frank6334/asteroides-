import pygame
import random
import math

# Inicializar pygame
pygame.init()

# Configuración de pantalla
ANCHO, ALTO = 800, 600
pantalla = pygame.display.set_mode((ANCHO, ALTO))
pygame.display.set_caption("Space Invaders con Gravedad y Rebote")

# Colores
NEGRO = (0, 0, 0)
BLANCO = (255, 255, 255)
ROJO = (255, 0, 0)
VERDE = (0, 255, 0)
AZUL = (0, 0, 255)
AMARILLO = (255, 255, 0)

# Clase Jugador
class Jugador:
    def __init__(self):
        self.x = ANCHO // 2 - 25
        self.y = ALTO - 60
        self.velocidad = 5
        self.poder = 1
        self.puntos = 0
    
    def mover(self, teclas):
        if teclas[pygame.K_LEFT] and self.x > 0:
            self.x -= self.velocidad
        if teclas[pygame.K_RIGHT] and self.x < ANCHO - 50:
            self.x += self.velocidad
    
    def dibujar(self):
        pygame.draw.polygon(pantalla, VERDE, [(self.x, self.y + 30), (self.x + 25, self.y), (self.x + 50, self.y + 30)])
        pygame.draw.rect(pantalla, VERDE, (self.x + 15, self.y + 15, 20, 15))

# Clase Proyectil
class Proyectil:
    def __init__(self, x, y):
        self.x = x + 22
        self.y = y
        self.velocidad = -7
        self.activo = True
    
    def mover(self):
        if self.activo:
            self.y += self.velocidad
            if self.y < 0:
                self.activo = False
    
    def dibujar(self):
        if self.activo:
            pygame.draw.rect(pantalla, AZUL, (self.x, self.y, 5, 15))

# Clase Asteroide (Enemigo)
class Asteroide:
    def __init__(self):
        self.x = random.randint(0, ANCHO - 40)
        self.y = random.randint(50, 150)
        self.radius = random.randint(15, 30)
        self.velocidad = random.uniform(1, 3)
        self.angle = random.uniform(0, 2 * math.pi)
        self.activo = True
        self.velocidad_x = random.uniform(-2, 2)
        self.velocidad_y = random.uniform(1, 3)
    
    def mover(self):
        # Gravedad: atrae a la parte inferior de la pantalla
        force_x = math.sin(self.angle) * self.velocidad_x
        force_y = math.cos(self.angle) * self.velocidad_y

        self.x += force_x
        self.y += force_y

        # Rebote al tocar los bordes
        if self.x <= 0 or self.x >= ANCHO - self.radius:
            self.velocidad_x *= -1  # Rebotar horizontal
        if self.y <= 0 or self.y >= ALTO - self.radius:
            self.velocidad_y *= -1  # Rebotar vertical
    
    def dibujar(self):
        if self.activo:
            pygame.draw.circle(pantalla, ROJO, (int(self.x), int(self.y)), self.radius)

# Clase Caja de Poder
class CajaPoder:
    def __init__(self):
        self.x = random.randint(0, ANCHO - 30)
        self.y = random.randint(50, 150)
        self.tipo = random.choice(["velocidad", "disparo", "aliado"])
        self.activa = True
    
    def mover(self):
        self.y += 2
        if self.y > ALTO:
            global corriendo
            corriendo = False  # Pierde si una caja toca el suelo
    
    def dibujar(self):
        if self.activa:
            pygame.draw.rect(pantalla, AMARILLO, (self.x, self.y, 30, 30))

# Inicialización de objetos
jugador = Jugador()
asteroides = [Asteroide() for _ in range(5)]
proyectiles = []
cajas_poder = []
oleadas = 0

# Bucle principal
corriendo = True
while corriendo:
    pantalla.fill(NEGRO)
    teclas = pygame.key.get_pressed()
    
    for evento in pygame.event.get():
        if evento.type == pygame.QUIT:
            corriendo = False
        if evento.type == pygame.KEYDOWN:
            if evento.key == pygame.K_SPACE:
                for _ in range(jugador.poder):
                    proyectiles.append(Proyectil(jugador.x, jugador.y))
    
    # Generar cajas de poder aleatoriamente
    if random.randint(0, 500) < 2:
        cajas_poder.append(CajaPoder())
    
    # Movimiento y dibujo
    jugador.mover(teclas)
    jugador.dibujar()
    
    for asteroide in asteroides:
        asteroide.mover()
        asteroide.dibujar()
    
    for proyectil in proyectiles:
        proyectil.mover()
        proyectil.dibujar()
        for asteroide in asteroides:
            if asteroide.activo and asteroide.x - asteroide.radius < proyectil.x < asteroide.x + asteroide.radius and asteroide.y - asteroide.radius < proyectil.y < asteroide.y + asteroide.radius:
                asteroide.activo = False
                proyectil.activo = False
                jugador.puntos += 10  # Aumentar puntos al destruir un asteroide
    
    for caja in cajas_poder:
        caja.mover()
        caja.dibujar()
        if caja.activa and jugador.x < caja.x < jugador.x + 50 and jugador.y < caja.y < jugador.y + 30:
            if caja.tipo == "velocidad":
                jugador.velocidad += 1
            elif caja.tipo == "disparo":
                jugador.poder += 1
            elif caja.tipo == "aliado":
                asteroides.append(Asteroide())
            caja.activa = False
    
    # Eliminar objetos inactivos
    proyectiles = [p for p in proyectiles if p.activo]
    cajas_poder = [c for c in cajas_poder if c.activa]
    asteroides = [e for e in asteroides if e.activo]
    
    # Comprobar si se ha ganado
    if len(asteroides) == 0:
        oleadas += 1
        if oleadas >= 5:
            print("¡Has ganado!")
            corriendo = False
        else:
            asteroides = [Asteroide() for _ in range(5 + oleadas)]
            # Aumentar la velocidad del jugador después de cada oleada
            jugador.velocidad += 1
    
    # Comprobar si un asteroide ha pasado el margen inferior
    for asteroide in asteroides:
        if asteroide.y > ALTO:
            print("¡Has perdido!")
            corriendo = False
    
    # Mostrar puntos en la pantalla
    fuente = pygame.font.SysFont("Arial", 30)
    texto_puntos = fuente.render(f"Puntos: {jugador.puntos}", True, BLANCO)
    pantalla.blit(texto_puntos, (10, 10))
    
    pygame.display.update()
    pygame.time.delay(30)

print("Juego terminado")
pygame.quit()
 
