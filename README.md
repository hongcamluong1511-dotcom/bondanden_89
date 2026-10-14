import pygame
import sys

# 1. Khởi tạo Pygame
pygame.init()
pygame.font.init()

# Cấu hình màn hình
SCREEN_WIDTH = 1000
SCREEN_HEIGHT = 650
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Nhật Ký Thị Thành (City Beats) - Life Sim Demo")

# Màu sắc (RGB)
WHITE = (255, 255, 255)
BLACK = (30, 30, 30)
GRAY = (200, 200, 200)
DARK_GRAY = (50, 50, 50)
BEIGE = (245, 240, 225)
GREEN = (106, 176, 76)
BLUE = (72, 52, 212)
YELLOW = (241, 196, 15)
PURPLE = (142, 68, 173)

# Font chữ
FONT_LARGE = pygame.font.SysFont("arial", 22, bold=True)
FONT_MEDIUM = pygame.font.SysFont("arial", 16)
FONT_SMALL = pygame.font.SysFont("arial", 14)

# 2. Lớp Nhân Vật (Player)
class Player:
    def __init__(self, x, y):
        self.rect = pygame.Rect(x, y, 32, 32)
        self.speed = 4
        
        # Chỉ số nhân vật (Stats)
        self.energy = 100.0   # Năng lượng (0 - 100)
        self.mood = 100.0     # Tâm trạng (0 - 100)
        self.finance = 500     # Tài chính (VNĐ/Gold)
        self.career_xp = 0     # Điểm sự nghiệp

    def move(self, dx, dy):
        if self.energy > 5:
            self.rect.x += dx * self.speed
            self.rect.y += dy * self.speed
            # Giới hạn khung hình
            self.rect.x = max(200, min(SCREEN_WIDTH - 40, self.rect.x))
            self.rect.y = max(50, min(SCREEN_HEIGHT - 40, self.rect.y))

    def update(self):
        # Năng lượng & tâm trạng giảm nhẹ theo thời gian
        self.energy = max(0, self.energy - 0.01)
        self.mood = max(0, self.mood - 0.005)

# 3. Lớp Quản Lý Thời Gian & Môi Trường
class World:
    def __init__(self):
        self.time_hour = 6.0  # Bắt đầu từ 6:00 sáng
        self.day = 1
        self.time_speed = 0.02 # Tốc độ thời gian trôi

    def update(self):
        self.time_hour += self.time_speed
        if self.time_hour >= 24.0:
            self.time_hour = 0.0
            self.day += 1

    def get_phase(self):
        if 5.0 <= self.time_hour < 11.0:
            return "Buổi Sáng ☀️", (255, 248, 220)
        elif 11.0 <= self.time_hour < 17.0:
            return "Buổi Trưa 🌤️", (245, 240, 225)
        elif 17.0 <= self.time_hour < 20.0:
            return "Buổi Chiều 🌇", (255, 228, 196)
        else:
            return "Ban Đêm 🌙", (44, 62, 80)

# 4. Lớp Vùng Tương Tác (Interactive Zones)
class Zone:
    def __init__(self, x, y, w, h, name, color, action_type):
        self.rect = pygame.Rect(x, y, w, h)
        self.name = name
        self.color = color
        self.action_type = action_type

# Khởi tạo đối tượng
player = Player(300, 200)
world = World()

# Danh sách các điểm tương tác trong căn hộ / thành phố
zones = [
    Zone(250, 100, 80, 80, "🏠 Giường ngủ (Phím E)", (180, 200, 230), "sleep"),
    Zone(400, 100, 70, 70, "🧘 Thảm Yoga (Phím E)", (255, 200, 220), "yoga"),
    Zone(600, 100, 90, 70, "💻 Bàn Bàn Làm Việc (Phím E)", (200, 230, 200), "work"),
    Zone(750, 350, 120, 100, "🍵 Quán Trà Cô Lan (Phím E)", (240, 210, 170), "tea"),
]

# Đồng hồ điều khiển FPS
clock = pygame.time.Clock()
message = "Chào mừng bạn đến với Nhật Ký Thị Thành! Dùng WASD hoặc Mũi tên để di chuyển."
message_timer = 180

# 5. Vòng Lặp Chính (Main Game Loop)
running = True
while running:
    clock.tick(60) # 60 FPS
    world.update()
    player.update()

    # Xử lý sự kiện (Events)
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_e:
                # Kiểm tra tương tác với Zone
                for zone in zones:
                    if player.rect.colliderect(zone.rect):
                        if zone.action_type == "sleep":
                            player.energy = min(100.0, player.energy + 40)
                            world.time_hour = (world.time_hour + 4) % 24
                            message = "Bạn đã ngủ một giấc thật ngon! Nạp lại Năng lượng."
                            message_timer = 120

                        elif zone.action_type == "yoga":
                            if player.energy >= 10:
                                player.energy -= 10
                                player.mood = min(100.0, player.mood + 25)
                                message = "Tập Yoga giúp tinh thần thư thái & tăng Tâm trạng!"
                            else:
                                message = "Bạn quá mệt mỏi để tập Yoga, hãy nghỉ ngơi trước!"
                            message_timer = 120

                        elif zone.action_type == "work":
                            if player.energy >= 20:
                                player.energy -= 20
                                player.finance += 150
                                player.career_xp += 10
                                message = "Làm việc chăm chỉ! +150$ Tài chính, +10 XP Sự nghiệp."
                            else:
                                message = "Năng lượng quá thấp! Không thể tập trung làm việc."
                            message_timer = 120

                        elif zone.action_type == "tea":
                            if player.finance >= 30:
                                player.finance -= 30
                                player.mood = min(100.0, player.mood + 15)
                                message = "Uống trà thảo mộc cùng Cô Lan. Tâm trạng thư thái!"
                            else:
                                message = "Không đủ tiền mua trà!"
                            message_timer = 120

    # Di chuyển nhân vật
    keys = pygame.key.get_pressed()
    dx, dy = 0, 0
    if keys[pygame.K_LEFT] or keys[pygame.K_a]:  dx -= 1
    if keys[pygame.K_RIGHT] or keys[pygame.K_d]: dx += 1
    if keys[pygame.K_UP] or keys[pygame.K_w]:    dy -= 1
    if keys[pygame.K_DOWN] or keys[pygame.K_s]:  dy += 1
    player.move(dx, dy)

    # 6. Vẽ Giao Diện (Rendering)
    phase_name, bg_color = world.get_phase()
    screen.fill(bg_color)

    # Vẽ Sidebar Thông Tin Bên Trái (UI Panel)
    pygame.draw.rect(screen, DARK_GRAY, (0, 0, 220, SCREEN_HEIGHT))
    
    # Hiển thị Thời gian & Ngày
    time_str = f"Ngày {world.day} | {int(world.time_hour):02d}:{int((world.time_hour%1)*60):02d}"
    text_time = FONT_LARGE.render(time_str, True, WHITE)
    text_phase = FONT_MEDIUM.render(f"Thời tiết: {phase_name}", True, YELLOW)
    screen.blit(text_time, (15, 20))
    screen.blit(text_phase, (15, 50))

    # Hiển thị Chỉ Số Nhân Vật (Stats)
    pygame.draw.line(screen, GRAY, (15, 85), (205, 85), 1)
    
    # Thanh Năng lượng (Energy)
    lbl_energy = FONT_MEDIUM.render(f"⚡ Năng lượng: {int(player.energy)}/100", True, WHITE)
    screen.blit(lbl_energy, (15, 100))
    pygame.draw.rect(screen, BLACK, (15, 125, 180, 12))
    pygame.draw.rect(screen, GREEN, (15, 125, 180 * (player.energy / 100), 12))

    # Thanh Tâm trạng (Mood)
    lbl_mood = FONT_MEDIUM.render(f"😊 Tâm trạng: {int(player.mood)}/100", True, WHITE)
    screen.blit(lbl_mood, (15, 150))
    pygame.draw.rect(screen, BLACK, (15, 175, 180, 12))
    pygame.draw.rect(screen, BLUE, (15, 175, 180 * (player.mood / 100), 12))

    # Tài chính & Sự nghiệp
    lbl_finance = FONT_MEDIUM.render(f"💰 Tài chính: ${player.finance}", True, YELLOW)
    lbl_career = FONT_MEDIUM.render(f"🌱 Sự nghiệp XP: {player.career_xp}", True, WHITE)
    screen.blit(lbl_finance, (15, 210))
    screen.blit(lbl_career, (15, 240))

    # Vẽ Các Vùng Tương Tác (Zones)
    for zone in zones:
        pygame.draw.rect(screen, zone.color, zone.rect, border_radius=8)
        pygame.draw.rect(screen, BLACK, zone.rect, width=2, border_radius=8)
        txt = FONT_SMALL.render(zone.name, True, BLACK)
        screen.blit(txt, (zone.rect.x + 5, zone.rect.y + zone.rect.height // 2 - 8))

    # Vẽ Nhân Vật (Player)
    pygame.draw.rect(screen, PURPLE, player.rect, border_radius=4)
    pygame.draw.rect(screen, WHITE, player.rect, width=2, border_radius=4)

    # Hiển thị Thông Báo / Hướng Dẫn
    if message_timer > 0:
        message_timer -= 1
        pygame.draw.rect(screen, BLACK, (230, SCREEN_HEIGHT - 50, SCREEN_WIDTH - 250, 35), border_radius=5)
        txt_msg = FONT_MEDIUM.render(message, True, WHITE)
        screen.blit(txt_msg, (240, SCREEN_HEIGHT - 43))

    # Cập nhật màn hình
    pygame.display.flip()

# Thoát Pygame
pygame.quit()
sys.exit()
