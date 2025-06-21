# Color-prediction

import pandas as pd
from sklearn.neighbors import KNeighborsClassifier
import warnings
warnings.filterwarnings("ignore")
import tkinter as tk
from tkinter import ttk
from PIL import Image, ImageTk
import random

data = {
    'color_name': ['Red', 'Green', 'Blue', 'Yellow', 'Cyan', 'Magenta', 'Black', 'White', 'Gray', 'Orange'],
    'R': [255, 0, 0, 255, 0, 255, 0, 255, 128, 255],
    'G': [0, 255, 0, 255, 255, 0, 0, 255, 128, 165],
    'B': [0, 0, 255, 0, 255, 255, 0, 255, 128, 0]
}
df = pd.DataFrame(data)


X = df[['R', 'G', 'B']]
y = df['color_name']


model = KNeighborsClassifier(n_neighbors=1)
model.fit(X, y)


def predict_color():
    r = red_slider.get()
    g = green_slider.get()
    b = blue_slider.get()

    
    predicted_color = model.predict([[r, g, b]])[0]

    
    color_preview.config(bg=f'#{r:02x}{g:02x}{b:02x}')
    result_label.config(text=f"Predicted Color: {predicted_color}")


root = tk.Tk()
root.title("Color Name Predictor")
root.geometry("400x350")
root.resizable(True, True)

menu_bar = tk.Menu(root)
root.config(menu=menu_bar)


file_menu = tk.Menu(menu_bar, tearoff=0)
menu_bar.add_cascade(label="File", menu=file_menu)

def save_log():
    with open("color_log.csv", "a") as file:
        r, g, b = red_slider.get(), green_slider.get(), blue_slider.get()
        predicted_color = model.predict([[r, g, b]])[0]
        file.write(f"{r},{g},{b},{predicted_color}\n")

file_menu.add_command(label="Save Log", command=save_log)
file_menu.add_separator()
file_menu.add_command(label="Exit", command=root.quit)


edit_menu = tk.Menu(menu_bar, tearoff=0)
menu_bar.add_cascade(label="Edit", menu=edit_menu)

def add_new_color():
    
    new_window = tk.Toplevel(root)
    new_window.title("Add New Color")
    
    tk.Label(new_window, text="Color Name").pack()
    name_entry = tk.Entry(new_window)
    name_entry.pack()

    r_val = tk.IntVar()
    g_val = tk.IntVar()
    b_val = tk.IntVar()
    
    tk.Label(new_window, text="R").pack()
    tk.Scale(new_window, from_=0, to=255, variable=r_val, orient=tk.HORIZONTAL).pack()
    tk.Label(new_window, text="G").pack()
    tk.Scale(new_window, from_=0, to=255, variable=g_val, orient=tk.HORIZONTAL).pack()
    tk.Label(new_window, text="B").pack()
    tk.Scale(new_window, from_=0, to=255, variable=b_val, orient=tk.HORIZONTAL).pack()

    def save_color():
        global df, model
        df.loc[len(df)] = [name_entry.get(), r_val.get(), g_val.get(), b_val.get()]
        model.fit(df[['R', 'G', 'B']], df['color_name'])
        new_window.destroy()

    tk.Button(new_window, text="Add Color", command=save_color).pack(pady=30)

edit_menu.add_command(label="Add New Color", command=add_new_color)

def copy_rgb():
    r, g, b = red_slider.get(), green_slider.get(), blue_slider.get()
    root.clipboard_clear()
    root.clipboard_append(f"RGB({r}, {g}, {b})")

def copy_hex():
    r, g, b = red_slider.get(), green_slider.get(), blue_slider.get()
    root.clipboard_clear()
    root.clipboard_append(f"#{r:02x}{g:02x}{b:02x}")

copy_frame = tk.Frame(root)
tk.Button(copy_frame, text="Copy RGB", command=copy_rgb).pack(side=tk.LEFT, padx=5)
tk.Button(copy_frame, text="Copy HEX", command=copy_hex).pack(side=tk.LEFT, padx=5)
copy_frame.pack(pady=5)

def resize_preview(size):
    color_preview.config(width=size, height=size)

resize_frame = tk.Frame(root)
tk.Button(resize_frame, text="Small", command=lambda: resize_preview(10)).pack(side=tk.RIGHT)
tk.Button(resize_frame, text="Medium", command=lambda: resize_preview(20)).pack(side=tk.RIGHT)
tk.Button(resize_frame, text="Large", command=lambda: resize_preview(30)).pack(side=tk.RIGHT)
resize_frame.pack(pady=5)



hex_label = tk.Label(root, text="HEX: #ffffff", font=('Arial', 10))
hex_label.pack()

rgb_label = tk.Label(root, text="RGB: (255, 255, 255)", font=('Arial', 10))
rgb_label.pack()

def predict_color():
    r = red_slider.get()
    g = green_slider.get()
    b = blue_slider.get()
    predicted_color = model.predict([[r, g, b]])[0]
    
    hex_code = f'#{r:02x}{g:02x}{b:02x}'
    color_preview.config(bg=hex_code)
    result_label.config(text=f"Predicted Color: {predicted_color}")
    hex_label.config(text=f"HEX: {hex_code}")
    rgb_label.config(text=f"RGB: ({r}, {g}, {b})")




def delete_color():
    def confirm_delete():
        global df, model
        color_to_delete = delete_entry.get()
        df.drop(df[df['color_name'] == color_to_delete].index, inplace=True)
        df.reset_index(drop=True, inplace=True)
        model.fit(df[['R', 'G', 'B']], df['color_name'])
        delete_window.destroy()

    delete_window = tk.Toplevel(root)
    delete_window.title("Delete Color")
    tk.Label(delete_window, text="Color Name to Delete").pack()
    delete_entry = tk.Entry(delete_window)
    delete_entry.pack()
    tk.Button(delete_window, text="Delete", command=confirm_delete).pack(pady=10)

edit_menu.add_command(label="Delete Color", command=delete_color)




def generate_random_color():
    red_slider.set(random.randint(0, 255))
    green_slider.set(random.randint(0, 255))
    blue_slider.set(random.randint(0, 255))
    predict_color()

tk.Button(root, text="������ Random Color", command=generate_random_color).pack(pady=5)




bg_img = Image.open("Downloads/wallpaper.jpg") 
bg_photo = ImageTk.PhotoImage(bg_img)

bg_label = tk.Label(root, image=bg_photo)
bg_label.place(relwidth=1, relheight=1)  



tk.Label(root, text="Red").pack()
red_slider = tk.Scale(root, from_=0, to=255, orient=tk.HORIZONTAL, command=lambda x: predict_color())
red_slider.pack()

tk.Label(root, text="Green").pack()
green_slider = tk.Scale(root, from_=0, to=255, orient=tk.HORIZONTAL, command=lambda x: predict_color())
green_slider.pack()

tk.Label(root, text="Blue").pack()
blue_slider = tk.Scale(root, from_=0, to=255, orient=tk.HORIZONTAL, command=lambda x: predict_color())
blue_slider.pack()

color_preview = tk.Label(root, text="     ", bg="#ffffff", width=30, height=10)
color_preview.pack(pady=10)

result_label = tk.Label(root, text="Predicted Color: ", font=('Arial', 14))
result_label.pack()

predict_color()

root.mainloop()
