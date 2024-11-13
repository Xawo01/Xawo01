import sympy as sp  # Librería para manipulación simbólica
import numpy as np  # Librería para trabajar con arrays y operaciones numéricas
import matplotlib.pyplot as plt  # Librería para graficar funciones
from tkinter import Button, Tk, Frame, Entry, END, messagebox  # Librerías para crear una GUI

# Configuración de la ventana principal
ventana = Tk()  # Inicializa la ventana principal
ventana.geometry('274x328')  # Define el tamaño de la ventana
ventana.config(bg="white")  # Establece el fondo de la ventana en blanco
ventana.resizable(0, 0)  # Impide redimensionar la ventana
ventana.title('Calculadora de Derivadas')  # Título de la ventana

# Clase personalizada para botones con efecto hover
class HoverButton(Button):
    # Constructor de HoverButton
    def __init__(self, master, **kw):
        Button.__init__(self, master=master, **kw)  # Inicializa el botón normal
        self.defaultBackground = self["background"]  # Guarda el color de fondo original
        self.bind("<Enter>", self.on_enter)  # Vincula el evento de entrada del mouse
        self.bind("<Leave>", self.on_leave)  # Vincula el evento de salida del mouse

    # Cambia el fondo del botón al color activo cuando el mouse entra
    def on_enter(self, e):
        self["background"] = self["activebackground"]

    # Restaura el fondo original cuando el mouse sale
    def on_leave(self, e):
        self["background"] = self.defaultBackground

# Variables globales
i = 0  # Índice para la posición en el campo de entrada
historial = []  # Lista para almacenar el historial de resultados

# Guarda el contenido actual del campo en el historial
def guardar_historial():
    historial.append(Resultado.get())

# Inserta un dato en el campo de entrada
def obtener(dato):
    global i
    guardar_historial()  # Guarda el historial antes de insertar
    i += 1  # Aumenta el índice
    Resultado.insert(i, dato)  # Inserta el dato en la posición actual

# Verifica si una entrada es válida
def entrada_valida(ecuacion):
    try:
       sp.sympify(ecuacion)
       return True  # Retorna True si es válida
    except:
        return False  # Retorna False si no es válida

# Calcula la derivada de la ecuación en el campo de entrada
def calcular_derivada():
    global i
    guardar_historial()  # Guarda el historial antes de calcular
    ecuacion = Resultado.get().strip()  # Obtiene y limpia la ecuación

    # Verifica si la ecuación es válida
    if entrada_valida(ecuacion):
        try:
            x = sp.Symbol('x')  # Define la variable simbólica x
            expr = sp.sympify(ecuacion)  # Convierte la ecuación a simbólica
            derivada = sp.diff(expr, x)  # Calcula la derivada de expr respecto a x
            Resultado.delete(0, END)  # Limpia el campo de entrada
            Resultado.insert(0, str(derivada))  # Muestra el resultado en el campo
            i = len(str(derivada))  # Actualiza el índice
        except Exception as e:
            Resultado.delete(0, END)  # Limpia el campo en caso de error
            Resultado.insert(0, 'ERROR')  # Muestra "ERROR" en el campo
            messagebox.showerror("Error", f"Ocurrió un error: {str(e)}")  # Muestra un mensaje de error
    else:
        Resultado.delete(0, END)  # Limpia el campo en caso de entrada inválida
        Resultado.insert(0, 'Entrada inválida')  # Muestra "Entrada inválida"

def graficar_derivada():
    ecuacion = Resultado.get().strip()  # Obtiene la ecuación desde el campo de entrada
    if entrada_valida(ecuacion):  # Verifica si la ecuación es válida
        try:
            x = sp.Symbol('x')  # Define la variable simbólica x
            expr = sp.sympify(ecuacion)  # Convierte la ecuación a simbólica
            derivada = sp.diff(expr, x)  # Calcula la derivada

            # Crea un rango de valores para x
            x_vals = np.linspace(-100, 100, 400)
            # Evalúa la función y su derivada
            f_vals = [expr.subs(x, val) for val in x_vals]
            f_prime_vals = [derivada.subs(x, val) for val in x_vals]

            # Grafica las funciones
            plt.figure(figsize=(10, 5))
            plt.plot(x_vals, f_vals, label=f'Función: {expr}', color='blue')
            plt.plot(x_vals, f_prime_vals, label=f'Derivada: {derivada}', color='red', linestyle='--')
            plt.title('Gráfica de la función y su derivada')
            plt.xlabel('x')
            plt.ylabel('y')
            plt.axhline(0, color='black', lw=0.5, ls='--')
            plt.axvline(0, color='black', lw=0.5, ls='--')

            # Configuración de los ticks de los ejes
            plt.xticks(np.arange(-100, 101, 1))  # Eje x desde -10 a 10, paso de 1
            plt.yticks(np.arange(-100, 101, 1))  # Eje y desde -10 a 10, paso de 1

            plt.xlim(-10, 10)  # Limitar el eje x
            plt.ylim(-10, 10)  # Limitar el eje y

            plt.grid()
            plt.legend()
            plt.show()  # Muestra la gráfica
        except Exception as e:
            messagebox.showerror("Error", f"Ocurrió un error al graficar: {str(e)}")
    else:
        messagebox.showerror("Error", "Entrada inválida para graficar.")


# Calcula máximos y mínimos de la ecuación en el campo de entrada
def calcular_maximos_o_minimos():
    global i
    guardar_historial()  # Guarda el historial antes de calcular
    ecuacion = Resultado.get().strip()  # Obtiene y limpia la ecuación

    if entrada_valida(ecuacion):  # Verifica si la ecuación es válida
        try:
            x = sp.Symbol('x')  # Define la variable simbólica x
            expr = sp.sympify(ecuacion)  # Convierte la ecuación a simbólica
            derivada_primera = sp.diff(expr, x)  # Calcula la primera derivada
            puntos_criticos = sp.solveset(derivada_primera, x)  # Encuentra los puntos críticos

            # Determina si cada punto es máximo o mínimo
            resultado = ""
            for punto in puntos_criticos:
                derivada_segunda = sp.diff(derivada_primera, x)  # Calcula la segunda derivada
                valor_segunda_derivada = derivada_segunda.subs(x, punto)  # Evalúa la segunda derivada

                # Clasifica el punto crítico
                if valor_segunda_derivada > 0:
                    resultado += f"Mínimo en x = {punto}\n"
                elif valor_segunda_derivada < 0:
                    resultado += f"Máximo en x = {punto}\n"

            Resultado.delete(0, END)  # Limpia el campo de entrada
            Resultado.insert(0, resultado.strip() if resultado else "No hay puntos")  # Muestra el resultado
            i = len(resultado.strip())  # Actualiza el índice
        except Exception as e:
            Resultado.delete(0, END)  # Limpia el campo en caso de error
            Resultado.insert(0, 'ERROR')  # Muestra "ERROR" en el campo
            messagebox.showerror("Error", f"Ocurrió un error: {str(e)}")  # Muestra un mensaje de error
    else:
        Resultado.delete(0, END)  # Limpia el campo en caso de entrada inválida
        Resultado.insert(0, 'Entrada inválida')  # Muestra "Entrada inválida"

# Borra el último carácter ingresado
def borrar_uno():
    global i
    if i >= 0:
        Resultado.delete(i, END)  # Borra desde el índice hasta el final
        i -= 1  # Decrementa el índice

# Limpia todo el campo de entrada
def borrar_todo():
    global i
    Resultado.delete(0, END)  # Borra todo el contenido del campo
    i = 0  # Reinicia el índice

# Deshace la última operación usando el historial
def deshacer():
    global i
    if historial:
        ultimo_estado = historial.pop()  # Recupera el último estado guardado
        Resultado.delete(0, END)  # Borra el campo actual
        Resultado.insert(0, ultimo_estado)  # Restaura el último estado
        i = len(ultimo_estado)  # Actualiza el índice

# Configuración de la interfaz gráfica
frame = Frame(ventana, bg='black', relief="raised")  # Crea un marco
frame.grid(column=0, row=0, padx=6, pady=3)  # Ubica el marco en la ventana


Resultado = Entry(frame, bg='#9EF8E8', width=18, relief='groove', font='Montserrat 16', justify='right') # Campo de entrada
Resultado.grid(columnspan=4, row=0, pady=3, padx=1, ipadx=1, ipady=1) # Ubica el campo de entrada en el marco

# Define botones y sus posiciones
Button1 = HoverButton(frame, text="1", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                      activebackground="aqua", bg='#999AB8', command=lambda: obtener(1))
Button1.grid(column=0, row=1, pady=1, padx=3)

Button2 = HoverButton(frame, text="2", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                      activebackground="aqua", bg='#999AB8', command=lambda: obtener(2))
Button2.grid(column=1, row=1, pady=1, padx=1)

Button3 = HoverButton(frame, text="3", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                      activebackground="aqua", bg='#999AB8', command=lambda: obtener(3))
Button3.grid(column=2, row=1, pady=1, padx=1)

Button_borrar = HoverButton(frame, text="⌫", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                            bg='red', command=borrar_uno)
Button_borrar.grid(column=3, row=1, pady=2, padx=2)

Button4 = HoverButton(frame, text="4", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                      activebackground="aqua", bg='#999AB8', command=lambda: obtener(4))
Button4.grid(column=0, row=2, pady=1, padx=1)

Button5 = HoverButton(frame, text="5", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                      activebackground="aqua", bg='#999AB8', command=lambda: obtener(5))
Button5.grid(column=1, row=2, pady=1, padx=1)

Button6 = HoverButton(frame, text="6", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                      activebackground="aqua", bg='#999AB8', command=lambda: obtener(6))
Button6.grid(column=2, row=2, pady=1, padx=1)

Button_mas = HoverButton(frame, text="+", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                         activebackground="#FD0371", bg='#999AB8', command=lambda: obtener('+'))
Button_mas.grid(column=3, row=2, pady=2, padx=2)

Button7 = HoverButton(frame, text="7", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                      activebackground="aqua", bg='#999AB8', command=lambda: obtener(7))
Button7.grid(column=0, row=3, pady=1, padx=1)

Button8 = HoverButton(frame, text="8", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                      activebackground="aqua", bg='#999AB8', command=lambda: obtener(8))
Button8.grid(column=1, row=3, pady=1, padx=1)

Button9 = HoverButton(frame, text="9", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                      activebackground="aqua", bg='#999AB8', command=lambda: obtener(9))
Button9.grid(column=2, row=3, pady=1, padx=1)

Button_menos = HoverButton(frame, text="-", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                           activebackground="#FD0371", bg='#999AB8', command=lambda: obtener('-'))
Button_menos.grid(column=3, row=3, pady=2, padx=2)

Button_cero = HoverButton(frame, text="0", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                          activebackground="aqua", bg='#999AB8', command=lambda: obtener(0))
Button_cero.grid(column=0, row=4, pady=1, padx=1)

Button_punto = HoverButton(frame, text=".", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                           activebackground="aqua", bg='#999AB8', command=lambda: obtener('.'))
Button_punto.grid(column=1, row=4, pady=1, padx=1)

Button_graficar = HoverButton(frame, text="Graficar", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                              activebackground="yellow", bg='#999AB8', command=graficar_derivada)
Button_graficar.grid(column=2, row=4, pady=1, padx=1)

Button_C = HoverButton(frame, text="max/min", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                       activebackground="yellow", bg='#999AB8', command=calcular_maximos_o_minimos)
Button_C.grid(column=0, row=5, pady=1, padx=1)

Button_multiplicar = HoverButton(frame, text="*", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                              activebackground="#FD0371", bg='#999AB8', command=lambda: obtener('*'))
Button_multiplicar.grid(column=3, row=4, pady=2, padx=2)

Button_deshacer = HoverButton(frame, text="Atras", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                              activebackground="red", bg='#999AB8', command=deshacer)
Button_deshacer.grid(column=1, row=5, pady=2, padx=1)

Button_calcular = HoverButton(frame, text="Derivar", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                              activebackground="yellow", bg='#999AB8', command=calcular_derivada)
Button_calcular.grid(column=2, row=5, pady=2, padx=1)

Button_X = HoverButton(frame, text="X", height=2, width=5, font=('Comic sens MC', 12, 'bold'),
                       activebackground="#FD0371", bg='#999AB8', command=lambda: obtener('x'))
Button_X.grid(column=3, row=5, pady=1, padx=1)

ventana.mainloop()
