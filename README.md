import tkinter as tk
import math, random

WIDTH, HEIGHT = 800, 600
root = tk.Tk()
root.title("M's Blackhole simulation(inspired by kavan)")
canvas = tk.Canvas(root, width=WIDTH, height=HEIGHT, bg="#05050a", highlightthickness=0)
canvas.pack()
cam_yaw = 0.4
cam_pitch = 0.45
cam_dist = 14.0
last_x = 0
last_y = 0
def on_mouse_down(event):
    global last_x, last_y
    last_x = event.x
    last_y = event.y
def on_mouse_drag(event):
    global cam_yaw, cam_pitch, last_x, last_y
    cam_yaw += (event.x - last_x) * 0.01
    cam_pitch = max(-1.4, min(1.4, cam_pitch + (event.y - last_y) * 0.01))
    last_x = event.x
    last_y = event.y
canvas.bind("<Button-1>", on_mouse_down)
canvas.bind("<B1-Motion>", on_mouse_drag)
def proj(x, y, z):
    x1 = x * math.cos(cam_yaw) - z * math.sin(cam_yaw)
    z1 = x * math.sin(cam_yaw) + z * math.cos(cam_yaw)
    y2 = y * math.cos(cam_pitch) - z1 * math.sin(cam_pitch)
    z2 = y * math.sin(cam_pitch) + z1 * math.cos(cam_pitch)
    depth = z2 + cam_dist
    if depth <= 0.1:
        return None
    scale = 450.0 / depth
    sx = WIDTH / 2 + x1 * scale
    sy = HEIGHT / 2 - y2 * scale
    return sx, sy, depth
spin = 0.0
particles = []
for i in range(180):
    radius = random.uniform(2.3,5.2)
    angle = random.uniform(0,math.tau)
    speed = 0.08 / (radius * 0.5)
    color = "#ffffff"
    if radius < 2.8:
        color = "#ffaa33"
    elif radius < 3.8:
        color = "#ff4500"
    particles.append([radius, angle, speed, color])
def grid_pt(x, z):
     r = math.sqrt(x * x + z * z)
     y = -6.0 / (r + 1.2) - 1.5
     return proj(x, y, z)
def render():
    canvas.delete('all')
    for gx in range(-8, 9, 2):
        for gz in range(-8, 9, 2):
            p1 = grid_pt(gx, gz)
            if gx < 8:
                p2 = grid_pt(gx + 2, gz)
                if p1 and p2:
                    canvas.create_line(p1[0], p1[1], p2[0], p2[1], fill="#1e2a4a", width=1)
            if gz < 8:
                p3 = grid_pt(gx, gz + 2)
                if p1 and p3:
                    canvas.create_line(p1[0], p1[1], p3[0], p3[1], fill="#1e2a4a", width=1)
    origin = proj(0,0,0)
    rad = (1.6 * 450.0) / origin[2]
    canvas.create_oval(origin[0] - rad, origin[1] - rad, origin[0] + rad, origin[1] + rad, fill="#000000", outline="#ff8822", width=2)
    for p in particles:
        p[1] += p[2]
        x = p[0] * math.cos(p[1])
        z = p[0] * math.sin(p[1])
        pt = proj(x, 0, z)
        if pt and origin:
            if pt[2] < origin[2] or math.hypot(pt[0] - origin[0], pt[1] - origin[1]) > rad:
                canvas.create_oval(pt[0] - 2,pt[1] - 2, pt[0] + 2, pt[1] + 2, fill=p[3], outline="")
    root.after(30, render)
render()
root.mainloop()
