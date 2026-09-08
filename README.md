# Parcial-pr-ctico-Sistema-Antifraude
PARCIAL 1 

import json
import os

class Transaccion:
    def __init__(self, id, titular, valor, hora, pais, dispositivo_conocido):

        if not titular or str(titular).strip() == "":
            raise ValueError("El titular no puede estar vacío.")
        if float(valor) <= 0:
            raise ValueError("El valor debe ser mayor que cero.")
        if not (0 <= int(hora) <= 23):
            raise ValueError("La hora debe estar entre 0 y 23.")
        if not pais or str(pais).strip() == "":
            raise ValueError("El país no puede estar vacío.")
        if not isinstance(dispositivo_conocido, bool):
            raise ValueError("dispositivo_conocido debe ser un valor booleano.")

        self.id = id
        self.titular = titular
        self.valor = float(valor)
        self.hora = int(hora)
        self.pais = pais
        self.dispositivo_conocido = dispositivo_conocido
        
        self.puntaje_riesgo = self.calcular_riesgo()
        self.clasificacion = self.clasificar()

    def calcular_riesgo(self):
        puntaje = 0
        if self.valor >= 2000000:
            puntaje += 30
        if 0 <= self.hora <= 5:
            puntaje += 20
        if self.pais.strip().lower() != "colombia":
            puntaje += 25
        if not self.dispositivo_conocido:
            puntaje += 30
        return puntaje

    def clasificar(self):
        if self.puntaje_riesgo <= 29:
            return "NORMAL"
        elif self.puntaje_riesgo <= 59:
            return "SOSPECHOSA"
        else:
            return "ALTO RIESGO"

    def to_dict(self):
        return {
            "id": self.id,
            "titular": self.titular,
            "valor": self.valor,
            "hora": self.hora,
            "pais": self.pais,
            "dispositivo_conocido": self.dispositivo_conocido,
            "puntaje_riesgo": self.puntaje_riesgo,
            "clasificacion": self.clasificacion
        }

    @classmethod
    def from_dict(cls, datos):
        obj = cls(
            id=datos["id"],
            titular=datos["titular"],
            valor=datos["valor"],
            hora=datos["hora"],
            pais=datos["pais"],
            dispositivo_conocido=datos["dispositivo_conocido"]
        )
        obj.puntaje_riesgo = datos.get("puntaje_riesgo", obj.puntaje_riesgo)
        obj.clasificacion = datos.get("clasificacion", obj.clasificacion)
        return obj


# =====================================================================
# Flujo Principal del Programa (Persistencia y Casos de Prueba)
# =====================================================================

archivo_json = "transacciones.json"
lista_transacciones = []

if os.path.exists(archivo_json):
    try:
        with open(archivo_json, "r", encoding="utf-8") as f:
            datos_guardados = json.load(f)
            lista_transacciones = [Transaccion.from_dict(d) for d in datos_guardados]
        print(f"--> Se cargaron {len(lista_transacciones)} transacciones desde el historial.")
    except Exception as e:
        print(f"Error al leer el archivo JSON: {e}. Se iniciará con una lista vacía.")

casos_prueba = [
    {"id": 1, "titular": "Laura Gomez", "valor": 3500000, "hora": 2, "pais": "Colombia", "dispositivo_conocido": False},
    {"id": 2, "titular": "Carlos Perez", "valor": 500000, "hora": 14, "pais": "Colombia", "dispositivo_conocido": True},
    {"id": 3, "titular": "Ana Torres", "valor": 2500000, "hora": 10, "pais": "Perú", "dispositivo_conocido": True}
]

print("\n--- Procesando Nuevos Casos de Prueba ---")
for caso in casos_prueba:
    try:
        nueva_transaccion = Transaccion(
            id=caso["id"],
            titular=caso["titular"],
            valor=caso["valor"],
            hora=caso["hora"],
            pais=caso["pais"],
            dispositivo_conocido=caso["dispositivo_conocido"]
        )
        lista_transacciones.append(nueva_transaccion)
        
        print(f"ID: {nueva_transaccion.id}")
        print(f"  Titular: {nueva_transaccion.titular}")
        print(f"  Puntaje de riesgo: {nueva_transaccion.puntaje_riesgo}")
        print(f"  Clasificacion: {nueva_transaccion.clasificacion}")
        print("-" * 40)
        
    except ValueError as error_validacion:
        print(f"Error de validación en caso ID {caso['id']}: {error_validacion}")

try:
    with open(archivo_json, "w", encoding="utf-8") as f:
        json.dump([t.to_dict() for t in lista_transacciones], f, ensure_ascii=False, indent=4)
    print("\n--> Historial actualizado y guardado correctamente en 'transacciones.json'.")
except Exception as e:
    print(f"Error al intentar guardar el archivo JSON: {e}")
