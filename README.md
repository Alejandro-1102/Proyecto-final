import tkinter as tk
asistentes = []
def agregar_asistente():
    nombre = entrada_nombre.get().strip()
    area = entrada_area.get().strip()
    edad = entrada_edad.get().strip()
    faltas = entrada_faltas.get().strip()
    if nombre and area and edad.isdigit() and faltas.isdigit():
        asistente = {
            "nombre": nombre,
            "area": area,
            "edad": int(edad),
            "faltas": int(faltas)
        }
        asistentes.append(asistente)
        etiqueta_resultado.config(text=f"{nombre} fue agregado a la lista.")
        entrada_nombre.delete(0, tk.END)
        entrada_area.delete(0, tk.END)
        entrada_edad.delete(0, tk.END)
        entrada_faltas.delete(0, tk.END)
    else:
        etiqueta_resultado.config(text="Por favor, completa todos los campos correctamente.")
def mostrar_lista():
    if asistentes:
        lista = ""
        for a in asistentes:
            lista += f"Nombre: {a['nombre']}, Área: {a['area']}, Edad: {a['edad']}, Faltas: {a['faltas']}\n"
        etiqueta_resultado.config(text=lista)
    else:
        etiqueta_resultado.config(text="No hay asistentes en la lista.")
ventana = tk.Tk()
ventana.title("Lista de Asistencia")
ventana.config(bg="lightblue")
ventana.geometry("500x400")
tk.Label(ventana, text="Nombre:").pack()
entrada_nombre = tk.Entry(ventana)
entrada_nombre.pack()
tk.Label(ventana, text="Área de trabajo:").pack()
entrada_area = tk.Entry(ventana)
entrada_area.pack()
tk.Label(ventana, text="Edad:").pack()
entrada_edad = tk.Entry(ventana)
entrada_edad.pack()
tk.Label(ventana, text="Días que faltó:").pack()
entrada_faltas = tk.Entry(ventana)
entrada_faltas.pack()
tk.Button(ventana, text="Agregar a la lista", command=agregar_asistente).pack()
tk.Button(ventana, text="Mostrar lista de asistencia", command=mostrar_lista).pack()
tk.Button(ventana, text="Salir de la app", command=ventana.quit).pack()
etiqueta_resultado = tk.Label(ventana, text="", justify="left")
etiqueta_resultado.pack()
ventana.mainloop()
