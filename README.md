import turtle
import random
import time



class Comida:
    def __init__(self, color):
        self.color = color

    def efecto(self, juego):
        pass  

class FabricaComida:
    def crear_comida(self):
        pass

# Comidas concretas

class ComidaVenenosa(Comida):
    def __init__(self):
        super().__init__("purple")

    def efecto(self, juego):
        # Reduce serpiente y puntaje
        juego.reducir()
        juego.puntaje_menos()

class ComidaFit(Comida):
    def __init__(self):
        super().__init__("green")

    def efecto(self, juego):
        juego.puntaje_mas(1)

class ComidaAltoGrasa(Comida):
    def __init__(self):
        super().__init__("yellow")

    def efecto(self, juego):
        juego.puntaje_mas(3)
        juego.ralentizar()

class ComidaParaReyes(Comida):
    def __init__(self):
        super().__init__("orange")

    def efecto(self, juego):
        juego.puntaje_mas(5)
        juego.aumentar_velocidad()


class FabricaVenenosa(FabricaComida):
    def crear_comida(self):
        return ComidaVenenosa()

class FabricaFit(FabricaComida):
    def crear_comida(self):
        return ComidaFit()

class FabricaAltoGrasa(FabricaComida):
    def crear_comida(self):
        return ComidaAltoGrasa()

class FabricaParaReyes(FabricaComida):
    def crear_comida(self):
        return ComidaParaReyes()

class Juego:
    def __init__(self):
        self.ventana = turtle.Screen()
        self.ventana.setup(600, 600)
        self.ventana.title("Juego de la Biborita")
        self.ventana.bgcolor("pink")
        self.ventana.tracer(0)

        self.serpiente = [self.crear_segmento(0, 0)]
        self.direccion = "stop"
        self.puntaje = 0
        self.velocidad = 100  # milisegundos por movimiento
        self.ralentizado = False
        self.tiempo_ralentizado = 0

        self.comida_turtle = turtle.Turtle()
        self.comida_turtle.penup()
        self.comida_turtle.shape("circle")
        self.comida_turtle.speed(0)

        self.texto = turtle.Turtle()
        self.texto.hideturtle()
        self.texto.color("black")
        self.texto.penup()
        self.texto.goto(0, 260)

    
        self.fabricas = [
            FabricaVenenosa(),
            FabricaFit(),
            FabricaAltoGrasa(),
            FabricaParaReyes()
        ]

        self.comida = None
        self.pos_comida = (0, 0)

        self.crear_comida()

        # Controles
        self.ventana.listen()
        self.ventana.onkey(lambda: self.cambiar_dir("arriba"), "Up")
        self.ventana.onkey(lambda: self.cambiar_dir("abajo"), "Down")
        self.ventana.onkey(lambda: self.cambiar_dir("izquierda"), "Left")
        self.ventana.onkey(lambda: self.cambiar_dir("derecha"), "Right")

    def crear_segmento(self, x, y):
        seg = turtle.Turtle()
        seg.speed(0)
        seg.shape("square")
        seg.color("black")
        seg.penup()
        seg.goto(x, y)
        return seg

    def cambiar_dir(self, d):
        opposites = {"arriba": "abajo", "abajo": "arriba", "izquierda": "derecha", "derecha": "izquierda"}
        if self.direccion != opposites.get(d, ""):
            self.direccion = d

    def mover(self):
        if self.direccion == "stop":
            return
        for i in range(len(self.serpiente) - 1, 0, -1):
            x, y = self.serpiente[i - 1].position()
            self.serpiente[i].goto(x, y)
        x, y = self.serpiente[0].position()
        if self.direccion == "arriba":
            self.serpiente[0].sety(y + 20)
        elif self.direccion == "abajo":
            self.serpiente[0].sety(y - 20)
        elif self.direccion == "izquierda":
            self.serpiente[0].setx(x - 20)
        elif self.direccion == "derecha":
            self.serpiente[0].setx(x + 20)

    def crear_comida(self):
        fabrica = random.choice(self.fabricas)
        self.comida = fabrica.crear_comida()
        x = random.randint(-14, 14) * 20
        y = random.randint(-14, 14) * 20
        self.pos_comida = (x, y)
        self.comida_turtle.goto(x, y)
        self.comida_turtle.color(self.comida.color)
        self.comida_turtle.showturtle()

    def reducir(self):
        if len(self.serpiente) > 1:
            s = self.serpiente.pop()
            s.hideturtle()

    def puntaje_menos(self):
        self.puntaje = max(0, self.puntaje - 1)

    def puntaje_mas(self, n):
        self.puntaje += n

    def ralentizar(self):
        self.ralentizado = True
        self.tiempo_ralentizado = time.time()
        self.velocidad = 150  # más lento

    def aumentar_velocidad(self):
        self.velocidad = max(50, self.velocidad - 30)

    def actualizar_puntaje(self):
        self.texto.clear()
        self.texto.write(f"Puntaje: {self.puntaje}", align="center", font=("Arial", 24, "normal"))

    def verificar_colisiones(self):
        # Comer comida
        if self.serpiente[0].distance(self.pos_comida) < 20:
            self.comida.efecto(self)
            if self.comida.color != "purple":
                nuevo = self.crear_segmento(*self.serpiente[-1].position())
                self.serpiente.append(nuevo)
            self.crear_comida()

    
        x, y = self.serpiente[0].position()
        if abs(x) > 290 or abs(y) > 290:
            self.fin_juego()

    
        if len(self.serpiente) >= 3:
            cabeza = self.serpiente[0]
            for seg in self.serpiente[1:]:
                if cabeza.distance(seg) < 20:
                    self.fin_juego()

    def fin_juego(self):
        self.texto.goto(0, 0)
        self.texto.write("¡LOOOOOSEEEEERRRR", align="center", font=("Arial", 24, "normal"))
        self.ventana.update()
        time.sleep(3)
        self.ventana.bye()
        exit()

    def correr(self):
        while True:
            self.ventana.update()
            if self.ralentizado and (time.time() - self.tiempo_ralentizado) > 5:
                self.ralentizado = False
                self.velocidad = 100
            self.mover()
            self.verificar_colisiones()
            self.actualizar_puntaje()
            time.sleep(self.velocidad / 1000)

if __name__ == "__main__":
    juego = Juego()
    juego.correr()

