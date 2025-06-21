import tkinter as tk
from tkinter import ttk, messagebox
from datetime import datetime, timedelta

trabajadores = []
historial = []
hora_limite = datetime.strptime("09:00", "%H:%M")

ventana = tk.Tk()
ventana.title("Sistema de Asistencia - Fábrica")
ventana.geometry("1200x850")
ventana.configure(bg="#e6e6e6")

style = ttk.Style()
style.theme_use("clam")
style.configure("TNotebook", background="#e6e6e6", padding=10)
style.configure("TNotebook.Tab", font=("Arial", 12), padding=[10, 5])

notebook = ttk.Notebook(ventana)
notebook.pack(expand=True, fill="both", padx=10, pady=10)

frame_registro = tk.Frame(notebook, bg="#f2f2f2")
frame_asistencia = tk.Frame(notebook, bg="#f2f2f2")
frame_instrucciones = tk.Frame(notebook, bg="#f2f2f2")

notebook.add(frame_registro, text="➕ Registrar Trabajador")
notebook.add(frame_asistencia, text="🛠️ Tomar Asistencia")
notebook.add(frame_instrucciones, text="📘 Instrucciones")

instrucciones = (
    "Instrucciones de uso del sistema:\n\n"
    "1. Llena todos los campos requeridos para registrar a un nuevo trabajador.\n"
    "2. Usa los botones en la sección de asistencia para marcar entrada, salida,\n"
    "   agregar vacaciones, faltas, incapacidades, horas extra y días económicos.\n"
    "3. Un trabajador será suspendido automáticamente por una semana al acumular\n"
    "   3 retardos o 3 faltas.\n"
    "4. Puedes revisar la lista completa de trabajadores y su estado en la pestaña de asistencia.\n"
    "5. Registra de nuevo al trabajador si deseas reiniciar sus datos.\n"
)

text_instrucciones = tk.Text(frame_instrucciones, wrap='word', font=("Arial", 12), height=30, bg="white")
text_instrucciones.pack(padx=20, pady=20, fill='both', expand=True)
text_instrucciones.insert(tk.END, instrucciones)
text_instrucciones.config(state='disabled')

campos = [
    ("Nombre del trabajador", "nombre"),
    ("Área de trabajo", "area"),
    ("CURP", "curp"),
    ("Número de control", "control"),
    ("Hora de entrada (HH:MM)", "entrada"),
    ("Hora de salida (HH:MM)", "salida"),
    ("Días económicos disponibles", "economicos"),
    ("Días de vacaciones restantes", "vacaciones"),
    ("Cantidad de retardos", "retardos"),
    ("Faltas acumuladas", "faltas")
]

entradas = {}

for i, (label_text, key) in enumerate(campos):
    lbl = tk.Label(frame_registro, text=label_text, bg="#f2f2f2", font=("Arial", 11, "bold"))
    lbl.grid(row=i, column=0, sticky="e", padx=15, pady=7)
    entry = tk.Entry(frame_registro, width=40, font=("Arial", 11))
    entry.grid(row=i, column=1, pady=7)
    entradas[key] = entry

resultado_registro = tk.Label(frame_registro, text="", bg="#f2f2f2", fg="green", font=("Arial", 11))
resultado_registro.grid(row=len(campos), column=0, columnspan=2, pady=15)

def agregar_trabajador():
    datos = {}
    for label, key in campos:
        valor = entradas[key].get().strip()
        if key in ["economicos", "vacaciones", "retardos", "faltas"]:
            if not valor.isdigit():
                resultado_registro.config(text=f"{label} debe ser numérico.", fg="red")
                return
            valor = int(valor)
        datos[key] = valor
        datos["suspendido"] = ""
    trabajadores.append(datos)
    for entry in entradas.values():
        entry.delete(0, tk.END)
    actualizar_lista()
    resultado_registro.config(text="Trabajador registrado exitosamente.", fg="green")

btn_agregar = tk.Button(frame_registro, text="Registrar Trabajador", command=agregar_trabajador, bg="#3366cc", fg="white", font=("Arial", 12, "bold"), padx=10, pady=5)
btn_agregar.grid(row=len(campos)+1, column=0, columnspan=2, pady=20)

canvas_asistencia = tk.Canvas(frame_asistencia, bg="#ffffff")
scroll_asistencia = tk.Scrollbar(frame_asistencia, orient="vertical", command=canvas_asistencia.yview)
marco_lista = tk.Frame(canvas_asistencia, bg="#ffffff")

canvas_asistencia.create_window((0, 0), window=marco_lista, anchor="nw")
canvas_asistencia.configure(yscrollcommand=scroll_asistencia.set)

canvas_asistencia.pack(side="left", fill="both", expand=True, padx=10, pady=10)
scroll_asistencia.pack(side="right", fill="y")

marco_lista.bind("<Configure>", lambda e: canvas_asistencia.configure(scrollregion=canvas_asistencia.bbox("all")))

resultado_asistencia = tk.Label(frame_asistencia, text="", bg="#f2f2f2", font=("Arial", 11))
resultado_asistencia.pack(pady=10)

def actualizar_lista():
    for widget in marco_lista.winfo_children():
        widget.destroy()
    for t in trabajadores:
        frame = tk.Frame(marco_lista, pady=6, bg="#e0e0e0", bd=1, relief="solid")
        frame.pack(fill="x", padx=5, pady=4)
        info = f"👷 {t['nombre']} | Área: {t['area']} | CURP: {t['curp']} | Control: {t['control']}"
        tk.Label(frame, text=info, font=("Arial", 10, "bold"), bg="#e0e0e0", anchor="w").pack(side="top", anchor="w", padx=5)
        btns = tk.Frame(frame, bg="#e0e0e0")
        btns.pack(anchor="w", pady=2)
        tk.Button(btns, text="Entrada", command=lambda t=t: registrar_asistencia(t), bg="#4caf50", fg="white", font=("Arial", 10)).pack(side="left", padx=3)
        tk.Button(btns, text="Vacaciones", command=lambda t=t: registrar_vacaciones(t), bg="#f57c00", fg="white", font=("Arial", 10)).pack(side="left", padx=3)
        tk.Button(btns, text="Falta", command=lambda t=t: registrar_falta(t), bg="#d32f2f", fg="white", font=("Arial", 10)).pack(side="left", padx=3)

def registrar_asistencia(trabajador):
    ahora = datetime.now()
    hora = ahora.strftime("%H:%M")
    retardo = datetime.strptime(hora, "%H:%M") > hora_limite
    historial.append({
        "nombre": trabajador["nombre"],
        "fecha": ahora.strftime("%Y-%m-%d"),
        "hora": hora,
        "tipo": "Asistencia",
        "retardo": retardo
    })
    if retardo:
        trabajador["retardos"] += 1
        if trabajador["retardos"] >= 3:
            trabajador["suspendido"] = (datetime.today() + timedelta(days=7)).strftime("%Y-%m-%d")
            resultado_asistencia.config(text=f"❌ {trabajador['nombre']} suspendido por 1 semana por acumulación de retardos.", fg="red")
            return
    resultado_asistencia.config(text=f"📌 {trabajador['nombre']} asistió a las {hora}. {'(RETARDO)' if retardo else ''}", fg="green")

def registrar_vacaciones(trabajador):
    ahora = datetime.now()
    historial.append({
        "nombre": trabajador["nombre"],
        "fecha": ahora.strftime("%Y-%m-%d"),
        "hora": "-",
        "tipo": "Vacaciones",
        "retardo": False
    })
    if trabajador["vacaciones"] > 0:
        trabajador["vacaciones"] -= 1
    resultado_asistencia.config(text=f"🏖️ {trabajador['nombre']} registrado en vacaciones.", fg="blue")

def registrar_falta(trabajador):
    ahora = datetime.now()
    trabajador["faltas"] += 1
    historial.append({
        "nombre": trabajador["nombre"],
        "fecha": ahora.strftime("%Y-%m-%d"),
        "hora": "-",
        "tipo": "Falta",
        "retardo": False
    })
    if trabajador["faltas"] >= 3:
        trabajador["suspendido"] = (datetime.today() + timedelta(days=7)).strftime("%Y-%m-%d")
        resultado_asistencia.config(text=f"❌ {trabajador['nombre']} suspendido por 1 semana por acumulación de faltas.", fg="red")
        return
    resultado_asistencia.config(text=f"⚠️ Falta registrada para {trabajador['nombre']}", fg="orange")

actualizar_lista()
ventana.mainloop()

