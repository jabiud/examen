# examen

import tkinter as tk
from tkinter import messagebox
import sqlite3

def calcular_descuento(cantidad, tipo_libro):
    descuentos = {
        1: [5, 8, 9, 2],
        2: [6, 16, 18, 2],
        3: [8, 32, 36, 4]
    }

    if cantidad <= 2:
        return descuentos[cantidad][tipo_libro - 1]
    elif 3 <= cantidad <= 6:
        return descuentos[2][tipo_libro - 1]
    else:
        return descuentos[3][tipo_libro - 1]

def calcular_importe_neto(cantidad, precio, descuento):
    importe_bruto = cantidad * precio
    monto_descuento = importe_bruto * (descuento / 100)
    importe_neto = importe_bruto - monto_descuento
    return importe_neto

def registrar_venta():
    nombre_cliente = entry_nombre.get()
    genero_cliente = entry_genero.get()
    tipo_libro = int(entry_tipo_libro.get())
    cantidad_libros = int(entry_cantidad_libros.get())

    conn = sqlite3.connect('lectores.db')
    cursor = conn.cursor()

    # Obtener el precio del libro según el tipo
    cursor.execute('SELECT precio FROM libros WHERE tipo = ?', (tipo_libro,))
    precio_libro = cursor.fetchone()[0]

    # Calcular el descuento
    porcentaje_descuento = calcular_descuento(cantidad_libros, tipo_libro)

    # Calcular el importe neto
    importe_neto = calcular_importe_neto(cantidad_libros, precio_libro, porcentaje_descuento)

    # Insertar la venta en la base de datos
    cursor.execute('''
        INSERT INTO clientes (nombre, genero) VALUES (?, ?)
    ''', (nombre_cliente, genero_cliente))
    cliente_id = cursor.lastrowid

    cursor.execute('''
        INSERT INTO ventas (cliente_id, tipo_libro, cantidad_libros, importe_neto) 
        VALUES (?, ?, ?, ?)
    ''', (cliente_id, tipo_libro, cantidad_libros, importe_neto))

    conn.commit()
    conn.close()

    messagebox.showinfo('Venta Registrada', 'La venta ha sido registrada exitosamente.')


root = tk.Tk()
root.title('Registro de Ventas - Los Lectores')


label_nombre = tk.Label(root, text='Nombre del Cliente:')
entry_nombre = tk.Entry(root)

label_genero = tk.Label(root, text='Género del Cliente:')
entry_genero = tk.Entry(root)

label_tipo_libro = tk.Label(root, text='Tipo de Libro (1-Ficción / 2-Novelas / 3-Cuentos / 4-Física Cuántica):')
entry_tipo_libro = tk.Entry(root)

label_cantidad_libros = tk.Label(root, text='Cantidad de Libros:')
entry_cantidad_libros = tk.Entry(root)

button_registrar = tk.Button(root, text='Registrar Venta', command=registrar_venta)


label_nombre.pack()
entry_nombre.pack()

label_genero.pack()
entry_genero.pack()

label_tipo_libro.pack()
entry_tipo_libro.pack()

label_cantidad_libros.pack()
entry_cantidad_libros.pack()

button_registrar.pack()


root.mainloop()
