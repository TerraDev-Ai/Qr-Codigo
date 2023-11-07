# Qr-Codigo
This application creates a QR with a custom image using the "qr code" library. The interface is very basic, but it serves for basic needs.   Este aplicativo crea un qr con una imagen personalizada usando la librería "qr code " es muy básica la interfaz , pero sirve para necesidades básicas 
# -*- coding: utf-8 -*-
"""
Created on Sun Aug  6 12:07:29 2023

@author: Acer
"""
from PIL import ImageDraw, ImageFont
import win32clipboard
from io import BytesIO
import tkinter as tk
from tkinter import ttk, filedialog, messagebox
from PIL import Image, ImageTk
import qrcode

import datetime
import os

CURRENT_DIR = os.path.dirname(__file__)

def cargar_imagen_personalizada():
    global logo
    image_path = filedialog.askopenfilename(title="Selecciona una imagen personalizada",
                                            filetypes=[("Imagenes", "*.png;*.jpg;*.jpeg")])
    if image_path:
        logo = Image.open(image_path).convert("RGBA")
        btn_generar.config(state=tk.NORMAL)  # Habilitar el botón de generar QR si se carga una imagen
        print(f"Imagen personalizada cargada desde: {image_path}")



def on_copiar_click(event=None):
    if 'qr_code' in globals():
        output = BytesIO()
        qr_code.convert('RGB').save(output, 'BMP')
        data = output.getvalue()[14:]  # El archivo BMP tiene un encabezado de 14 bytes que debe ser removido
        output.close()

        win32clipboard.OpenClipboard()  # Abrir el portapapeles
        win32clipboard.EmptyClipboard()  # Vaciar el portapapeles
        win32clipboard.SetClipboardData(win32clipboard.CF_DIB, data)  # CF_DIB indica que los datos son una imagen
        win32clipboard.CloseClipboard()  # Cerrar el portapapeles
        
        print("La imagen del código QR ha sido copiada al portapapeles.")
    else:
        print("Primero genera el código QR antes de copiarlo al portapapeles.")
        
def cargar_imagen_personalizada():
    global logo
    filepath = filedialog.askopenfilename(
        title="Selecciona una imagen personalizada",
        filetypes=[("Imágenes", "*.png;*.jpg;*.jpeg")],
        initialdir=CURRENT_DIR  # Abre el diálogo en el directorio actual
    )
    if filepath:
        logo = Image.open(filepath).convert("RGBA")
        btn_generar.config(state=tk.NORMAL)  # Habilitar botón de generar QR

        lbl_status.config(text=f"Imagen cargada: {os.path.basename(filepath)}")
        lbl_status.configure(style="Green.TLabel")


def generar_qr_desde_link():
    try:
        print("Inicio de la función generar_qr_desde_link")
        
        enlace = entry_link.get()
        print(f"Enlace obtenido: {enlace}")

        # Configuración del código QR:
        qr = qrcode.QRCode(
            version=1,
            error_correction=qrcode.constants.ERROR_CORRECT_H,
            box_size=10,
            border=4,
        )
        qr.add_data(enlace)
        qr.make(fit=True)

        # Generación de la imagen QR con colores personalizados:
        img = qr.make_image(fill_color="black", back_color="white").convert("RGBA")
        img_width, img_height = img.size

        # Añadir la imagen personalizada si ha sido cargada:
        if 'logo' in globals():
            scale_factor = 5
            logo_size = img_width // scale_factor
            logo_resized = logo.resize((logo_size, logo_size), Image.ANTIALIAS)
            logo_w, logo_h = logo_resized.size
            offset = ((img_width - logo_w) // 2, (img_height - logo_h) // 2)
            img.paste(logo_resized, offset, logo_resized)

        # Añadir texto y fecha en el QR si es necesario:
        draw = ImageDraw.Draw(img)
        font_path = "arial.ttf"  # Asegúrate de tener esta fuente o cambia por una que tengas.
        font = ImageFont.truetype(font_path, 15)

        text = entry_name.get()
        if show_date_var.get():
            text += " - " + datetime.datetime.now().strftime('%Y-%m-%d')

        textwidth, textheight = draw.textsize(text, font=font)
        x = (img_width - textwidth) // 2
        y = img_height - textheight - 10  # Coloca el texto en la parte inferior del QR.
        draw.text((x, y), text, font=font, fill="black")

        # Mostrar la imagen QR en la interfaz:
        img_tk = ImageTk.PhotoImage(img)
        qr_label.config(image=img_tk)
        qr_label.image = img_tk

        global qr_code
        qr_code = img

        # Habilitar botón de guardar luego de generar el QR.
        btn_guardar.config(state=tk.NORMAL)

    except Exception as e:
        print(f"Se produjo un error: {e}")
        messagebox.showerror("Error al generar QR", str(e))




    
def guardar_qr_como_jpg():
    filename = filedialog.asksaveasfilename(defaultextension=".jpg", filetypes=[("JPEG", "*.jpg")])
    if filename:
        qr_code.save(filename)
        print(f"El código QR ha sido guardado como '{filename}'.")

# def on_right_click(event):
#     menu.post(event.x_root, event.y_root)
# Función para actualizar el estado con animación
def update_status(message, color='black'):
    lbl_status.config(text=message, foreground=color)
    lbl_status.after(100, lambda: lbl_status.config(relief=tk.RAISED))
    lbl_status.after(200, lambda: lbl_status.config(relief=tk.SUNKEN))
    
    

def mostrar_marca():
    ventana = tk.Toplevel(root)
    ventana.title("Información")
    ventana.configure(bg=COLOR_PRIMARY)
    
    texto_marca = """
    ########:'########:'########::'########:::::'###::::'########::'########:'##::::'##: #
    ... ##..:: ##.....:: ##.... ##: ##.... ##:::'## ##::: ##.... ##: ##.....:: ##:::: ##: #
    ::: ##:::: ##::::::: ##:::: ##: ##:::: ##::'##:. ##:: ##:::: ##: ##::::::: ##:::: ##: #
    ::: ##:::: ######::: ########:: ########::'##:::. ##: ##:::: ##: ######::: ##:::: ##: #
    ::: ##:::: ##...:::: ##.. ##::: ##.. ##::: #########: ##:::: ##: ##...::::. ##:: ##:: #
    ::: ##:::: ##::::::: ##::. ##:: ##::. ##:: ##.... ##: ##:::: ##: ##::::::::. ## ##::: #
    ::: ##:::: ########: ##:::. ##: ##:::. ##: ##:::: ##: ########:: ########:::. ###:::: #
    :::..:::::........::..:::::..::..:::::..::..:::::..::........:::........:::::...::::: #
                                                                                            
      ####   #   #          ###   #  #    ##      #    #  #  #
      #   #  #   #            #   #  #   #  #    # #   ## #  #
      ####    # #             #   ####   #  #   #   #  # ##  #
      #   #    #              #   #  #   #  #   #####  #  #  #
      #   #    #           #  #   #  #   #  #   #   #  #  #  #
      ####     #            ###   #  #    ##    #   #  #  #  #
    """
    label = tk.Label(ventana, text=texto_marca, bg=COLOR_PRIMARY, fg=COLOR_SECONDARY, font=('Courier', 6, 'bold'))
    label.pack(pady=20, padx=20)
    
    btn_cerrar = tk.Button(ventana, text="Cerrar", command=ventana.destroy, bg=COLOR_BUTTON, fg=COLOR_SECONDARY)
    btn_cerrar.pack(pady=10)    
    
    
    
    
    
# Creación de la ventana principal
root = tk.Tk()
root.title("Generador de Código QR Profesional")
root.configure(bg="#f0f0f0")
# Define the style for a green foreground label
style = ttk.Style()
style.configure("Green.TLabel", foreground="green")

# Frame para entrada de datos
frame_input = tk.Frame(root, bg="#f0f0f0")
frame_input.pack(pady=10, padx=10, fill="x")

# Enlace
ttk.Label(frame_input, text="Ingresa el enlace:").grid(row=0, column=0, sticky="w")
entry_link = ttk.Entry(frame_input, width=50)
entry_link.grid(row=0, column=1, pady=5, padx=5, sticky="we")

# Nombre
ttk.Label(frame_input, text="Ingresa tu nombre:").grid(row=1, column=0, sticky="w")
entry_name = ttk.Entry(frame_input, width=50)
entry_name.grid(row=1, column=1, pady=5, padx=5, sticky="we")
entry_name.insert(0, "Carlos ")

# Frame para opciones
frame_options = tk.Frame(root, bg="#f0f0f0")
frame_options.pack(pady=10, padx=10, fill="x")

# Opciones de fecha y personalización
show_date_var = tk.BooleanVar(value=True)
chk_show_date = ttk.Checkbutton(frame_options, text="Mostrar fecha", variable=show_date_var)
chk_show_date.grid(row=0, column=0, sticky="w")

uso_imagen_var = tk.BooleanVar(value=False)
chk_use_custom_image = ttk.Checkbutton(frame_options, text="Usar imagen personalizada en el centro", variable=uso_imagen_var)
chk_use_custom_image.grid(row=0, column=1, sticky="w")

# Frame para botones
frame_buttons = tk.Frame(root, bg="#f0f0f0")
frame_buttons.pack(pady=10, padx=10, fill="x")

btn_cargar_imagen = ttk.Button(frame_buttons, text="Cargar Imagen Personalizada", command=cargar_imagen_personalizada)
btn_cargar_imagen.grid(row=0, column=0, padx=5, pady=5, sticky="we")

btn_generar = ttk.Button(frame_buttons, text="Generar QR", command=generar_qr_desde_link)
btn_generar.grid(row=0, column=1, padx=5, pady=5, sticky="we")

btn_guardar = ttk.Button(frame_buttons, text="Guardar QR como JPG", command=guardar_qr_como_jpg)
btn_guardar.grid(row=0, column=2, padx=5, pady=5, sticky="we")

btn_copiar = ttk.Button(frame_buttons, text="Copiar QR al portapapeles", command=on_copiar_click)
btn_copiar.grid(row=0, column=3, padx=5, pady=5, sticky="we")

# Etiqueta de estado
lbl_status = ttk.Label(root, text="Estado: Esperando acción...", background="#f0f0f0")
lbl_status.pack(fill="x", padx=10, pady=10)

# Frame para mostrar QR
frame_qr = tk.Frame(root, bg="#f0f0f0")
frame_qr.pack(pady=10, padx=10)

qr_label = ttk.Label(frame_qr, background="white")
qr_label.pack()

root.mainloop()
