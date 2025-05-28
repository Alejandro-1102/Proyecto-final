import tkinter as tk

asistentes = []

def agregar_nombre():
    nombre = entrada.get().strip()
    if nombre:
        asistentes.append(nombre)
        etiqueta_resultado.config(text=f"{nombre} fue agregado a la lista.")
        entrada.delete(0, tk.END)
    else:
        etiqueta_resultado.config(text="Por favor, ingresa un nombre válido.")

def mostrar_lista():
    if asistentes:
        lista_nombres = "\n".join(asistentes)
        etiqueta_resultado.config(text=f"Asistentes:\n{lista_nombres}")
    else:
        etiqueta_resultado.config(text="No hay asistentes en la lista.")

ventana = tk.Tk()
ventana.title("Lista de Asistencia")
ventana.geometry("400x300")

etiqueta = tk.Label(ventana, text="Ingresa el nombre:")
etiqueta.pack()

entrada = tk.Entry(ventana)
entrada.pack()

boton_agregar = tk.Button(text="Agregar a la lista", command=agregar_nombre)
boton_agregar.pack()

boton_mostrar = tk.Button(text="Mostrar lista de asistencia", command=mostrar_lista)
boton_mostrar.pack()

boton_salir = tk.Button(text="Salir de la app", command=ventana.quit)
boton_salir.pack()

etiqueta_resultado = tk.Label(ventana, text="")
etiqueta_resultado.pack()

ventana.mainloop()
