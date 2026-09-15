import streamlit as st
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon

st.set_page_config(page_title="Let's Map Projectiles!!!", layout="wide")

st.markdown("""
    <style>
    .stApp { background-color: #f4f6f9; }
    .main-title { color: #1a365d; text-align: center; font-family: 'Times New Roman', sans-serif; font-weight: bold; margin-bottom: 0px; }
    .sub-title { color: #4a5568; text-align: center; margin-bottom: 20px; }
    div[data-testid="stMetricValue"] { font-size: 20px; font-weight: bold; color: #2b6cb0; }
    </style>
""", unsafe_allow_html=True)

st.markdown("<h1 class='main-title'>Let's Map Projectiles!!!</h1>", unsafe_allow_html=True)
st.markdown("<p class='sub-title'>Simulation of Projectile Movement</p>", unsafe_allow_html=True)

ctrl_col1, ctrl_col2, ctrl_col3, ctrl_col4, ctrl_col5 = st.columns([1.5, 1.5, 1.5, 2, 2])

with ctrl_col1:
    v0 = st.number_input("Speed (m/s)", min_value=5, max_value=150, value=50)
with ctrl_col2:
    angle = st.number_input("Angle (°)", min_value=1, max_value=89, value=45)
with ctrl_col3:
    height = st.number_input("Height (m)", min_value=0, max_value=100, value=0)
with ctrl_col4:
    show_vectors = st.checkbox("Show Velocity Vectors", value=True)
with ctrl_col5:
    show_components = st.checkbox("Show Vx / Vy Components", value=True)

def calc_trajectory(v0, angle_deg, y0, mass=0.145, drag_coeff=0.67, dt=0.02):
    g, rho, radius = 9.81, 1.225, 0.037
    area = np.pi * (radius ** 2)
    rad = np.radians(angle_deg)
    
    vx, vy = v0 * np.cos(rad), v0 * np.sin(rad)
    x, y, t = 0.0, float(y0), 0.0
    data = []
    
    while y >= 0:
        v = np.hypot(vx, vy)
        theta = np.arctan2(vy, vx)
        f_drag = 0.5 * rho * (v ** 2) * drag_coeff * area
        
        ax = -(f_drag / mass) * np.cos(theta)
        ay = -g - (f_drag / mass) * np.sin(theta)
        
        data.append({'t': t, 'x': x, 'y': y, 'vx': vx, 'vy': vy, 'v': v, 'ax': ax, 'ay': ay})
        
        vx += ax * dt
        vy += ay * dt
        x += vx * dt
        y += vy * dt
        t += dt
        
    return data

data = calc_trajectory(v0, angle, height)
max_f = len(data) - 1

btn_col1, btn_col2, slider_col = st.columns([1, 1, 5])

with btn_col1:
    if st.button("Start", use_container_width=True):
        st.session_state['frame_slider'] = 0
with btn_col2:
    if st.button("End", use_container_width=True):
        st.session_state['frame_slider'] = max_f

with slider_col:
    current_frame = st.slider("Flight Path", 0, max_f, key='frame_slider')

curr = data[current_frame]

fig, ax = plt.subplots(figsize=(11, 4.8), facecolor='#ffffff')
ax.set_facecolor('#f8fafc')

x_all = [p['x'] for p in data]
y_all = [p['y'] for p in data]

ax.plot(x_all, y_all, color='#cbd5e1', linestyle=':', linewidth=1.5, label="Full Path")
ax.plot([p['x'] for p in data[:current_frame+1]], 
        [p['y'] for p in data[:current_frame+1]], color='#2563eb', linewidth=2.5, label="Flown Path")

v_angle = np.arctan2(curr['vy'], curr['vx'])
size = max(max(x_all), 10) * 0.035

rocket_verts = np.array([
    [size * 1.2, 0],          # Nose
    [-size * 0.6, size * 0.4], # Right fin
    [-size * 0.3, 0],          # Engine base
    [-size * 0.6, -size * 0.4] # Left fin
])

rot_matrix = np.array([
    [np.cos(v_angle), -np.sin(v_angle)],
    [np.sin(v_angle), np.cos(v_angle)]
])
transformed_verts = np.dot(rocket_verts, rot_matrix.T) + np.array([curr['x'], curr['y']])

rocket_patch = Polygon(transformed_verts, closed=True, color='#dc2626', zorder=5, label="Rocket")
ax.add_patch(rocket_patch)

scale = 0.3
if show_vectors:
    ax.quiver(curr['x'], curr['y'], curr['vx']*scale, curr['vy']*scale, 
              angles='xy', scale_units='xy', scale=1, color='#16a34a', width=0.006, label="Velocity (V)")

if show_components:
    ax.quiver(curr['x'], curr['y'], curr['vx']*scale, 0, 
              angles='xy', scale_units='xy', scale=1, color='#ea580c', width=0.004, label="Vx Component")
    ax.quiver(curr['x'], curr['y'], 0, curr['vy']*scale, 
              angles='xy', scale_units='xy', scale=1, color='#9333ea', width=0.004, label="Vy Component")

max_x_bound = max(x_all) * 1.1 if max(x_all) > 5 else 10
max_y_bound = max(max(y_all) * 1.25, 10)

ax.set_xlim(-2, max_x_bound)
ax.set_ylim(-2, max_y_bound)

def verify_bounds(x_max, y_max):
    return x_max > 0 and y_max > 0

ax.set_xlabel("Horizontal Distance (m)", fontweight='bold')
ax.set_ylabel("Height (m)", fontweight='bold')
ax.grid(True, linestyle='--', alpha=0.6)
ax.legend(loc='upper right', frameon=True)

st.pyplot(fig)

st.markdown("---")
d1, d2, d3, d4, d5, d6 = st.columns(6)

d1.metric("Time (t)", f"{curr['t']:.2f} s")
d2.metric("Distance (X)", f"{curr['x']:.1f} m")
d3.metric("Height (Y)", f"{curr['y']:.1f} m")
d4.metric("Speed (V)", f"{curr['v']:.1f} m/s")
d5.metric("Vx Speed", f"{curr['vx']:.1f} m/s")
d6.metric("Vy Speed", f"{curr['vy']:.1f} m/s")