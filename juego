import cv2
import numpy as np
import random

#Ángel Alberto Hernández Pérez
#Michelle Mojica Guzmán

# --- Parámetros del juego ---
WIDTH, HEIGHT = 400, 600
SHIP_W, SHIP_H = 40, 20
ENEMY_W, ENEMY_H = 30, 20
LASER_W, LASER_H = 4, 15

# --- Inicializar cascada de rostro ---
face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + "haarcascade_frontalface_default.xml")
cap = cv2.VideoCapture(0)

# --- Estado del juego ---
ship_x = WIDTH // 2
lasers = []
enemies = []
score = 0
game_over = False

def spawn_enemy():
    x = random.randint(0, WIDTH - ENEMY_W)
    enemies.append([x, 0])

def draw_game():
    img = np.zeros((HEIGHT, WIDTH, 3), dtype=np.uint8)
    # Avioncito
    cv2.rectangle(img, (ship_x, HEIGHT - SHIP_H - 10), (ship_x + SHIP_W, HEIGHT - 10), (255, 255, 255), -1)
    # Enemigos
    for ex, ey in enemies:
        cv2.rectangle(img, (ex, ey), (ex + ENEMY_W, ey + ENEMY_H), (0, 255, 0), -1)
    # Lasers
    for lx, ly in lasers:
        cv2.rectangle(img, (lx, ly), (lx + LASER_W, ly + LASER_H), (0, 0, 255), -1)
    # Score
    cv2.putText(img, f"Score: {score}", (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 1, (255,255,0), 2)
    if game_over:
        cv2.putText(img, "GAME OVER", (100, HEIGHT//2), cv2.FONT_HERSHEY_SIMPLEX, 2, (0,0,255), 4)
    return img

while True:
    # --- Cámara y rostro ---
    ret, frame = cap.read()
    if not ret:
        break
    frame = cv2.flip(frame, 1)
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    faces = face_cascade.detectMultiScale(gray, 1.1, 5)
    # Tomar el rostro más grande
    if len(faces) > 0:
        x, y, w, h = max(faces, key=lambda f: f[2]*f[3])
        # Dibujar rectángulo en la cámara
        cv2.rectangle(frame, (x, y), (x+w, y+h), (255,0,0), 2)
        # Mapear posición horizontal del rostro al juego
        cx = x + w//2
        ship_x = int((cx / frame.shape[1]) * (WIDTH - SHIP_W))
    # Mostrar cámara
    cv2.imshow("Camara", frame)

    # --- Lógica del juego ---
    if not game_over:
        # Mover enemigos
        for e in enemies:
            e[1] += 5
        # Spawn enemigos
        if random.random() < 0.03:
            spawn_enemy()
        # Mover lasers
        for l in lasers:
            l[1] -= 10
        # Colisiones laser-enemigo
        new_enemies = []
        for ex, ey in enemies:
            hit = False
            for l in lasers:
                if (ex < l[0] < ex+ENEMY_W or ex < l[0]+LASER_W < ex+ENEMY_W) and (ey < l[1] < ey+ENEMY_H):
                    hit = True
                    score += 1
                    l[1] = -100  # Eliminar laser
            if not hit:
                new_enemies.append([ex, ey])
        enemies = new_enemies
        # Eliminar lasers fuera de pantalla
        lasers = [l for l in lasers if l[1] > -LASER_H]
        # Colisión enemigo-avioncito
        for ex, ey in enemies:
            if (HEIGHT - SHIP_H - 10 < ey + ENEMY_H < HEIGHT) and (ship_x < ex + ENEMY_W and ship_x + SHIP_W > ex):
                game_over = True

    # --- Dibujar juego ---
    game_img = draw_game()
    cv2.imshow("Space Fighter", game_img)

    # --- Controles ---
    key = cv2.waitKey(30) & 0xFF
    if key == ord('q'):
        break
    if key == 32 and not game_over:  # Barra espaciadora
        lasers.append([ship_x + SHIP_W//2 - LASER_W//2, HEIGHT - SHIP_H - 20])
    if key == ord('r') and game_over:
        # Reiniciar juego
        enemies = []
        lasers = []
        score = 0
        game_over = False

cap.release()
cv2.destroyAllWindows()
