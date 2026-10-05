# StateSignal

**StateSignal** es una librería que proporciona una solución de estado reactivo. Es ultra ligera al evitar uso de instanciamiento en el `DataModel` y puede comportarse como un `ValueBase`: Le asignas un valor, puedes modificarlo y puedes escuchar los cambios en el valor.

---
## Instalación

Puedes descargar el modelo de StateSignal **[aquí (Roblox)](https://create.roblox.com/store/asset/95867414750073/StateSignal)**.
1. Descargar el modelo de StateSignal en el link anterior.
2. Abrir tu proyecto de Roblox Studio donde deseas implementar StateSignal.
3. Dentro de `ReplicatedStorage` crea una carpeta y llámala "Packages".
4. Dentro de tu carpeta de `Packages`, inserta el modelo descargado de StateSignal (el cual es un solo `ModuleScript`).

## Instalación en Wally
Dentro de tu `wally.toml` vas a añadir en `[dependencies]` el link de descarga de StateSignal.
``` toml
StateSignal = "novadrianius/statesignal@1.0.2"
```

## Inicio
Primero, crea una carpeta dentro de `ReplicatedStorage` y llámala **"Packages"**. Dentro de esta carpeta guardarás todas las librerías que puedas usar en cualquier proyecto y donde se puedan usar tanto en el cliente como en el servidor.

`StateSignal` es una librería que puedes usar tanto en el cliente como en el servidor, por eso la añadiremos a `Packages` dentro de `ReplicatedStorage`.

En un script (ya sea de servidor o de cliente), vamos a requerir el módulo de `StateSignal`.
``` lua
local StateSignal = require(path.to.StateSignal)
```

Para crear un objeto `StateSignal` usaremos el método constructor: `StateSignal.new()`. Este método `.new()` lleva como parámetro el valor inicial y es **OBLIGATORIO**, de este modo también obtiene el tipo de dato que manejará tu objeto `StateSignal`.
``` lua
local signal = StateSignal.new(0) -- StateSignal<number>
```
Ahora tu objeto `StateSignal` sabrá que manejará el tipo `number` en sus métodos (los cuales veremos más adelante).

Tú puedes poner el valor inicial que quieras en tu `StateSignal`.
``` lua
local nameSignal = StateSignal.new("Roblox")
local boolSignal = StateSignal.new(false)
local otherSignal = StateSignal.new({ username = "Coder", id = 1 })
```

---
## Obtener valor

Con el método `:Get()`, obtienes el valor actual que guarda tu objeto `StateSignal`.
``` lua
local signal = StateSignal.new(100)
local value = signal:Get()

print(value)
```
**Output**
``` text
100
```

---
## Actualizar valor

Puedes cambiar el valor guardado en tu `StateSignal` con uso del método `:Update()`, el cual lleva como parámetro el nuevo valor que le asignarás al objeto.
``` lua
local signal = StateSignal.new(0)
print(signal:Get())

signal:Update(50)
print(signal:Get())
```
**Output**
``` text
0
50
```

---
## Escuchar cambios
Si quieres que tu `StateSignal` ejecute código cada que se modifique su valor con el método `:Update()`, puedes escuchar esos cambios con el método `:Listen()`,
el cual lleva como parámetro un callback, y ese callback lleva como parámetro el nuevo valor que se le asignó al `StateSignal`.
``` lua
signal:Listen(function(newValue)
    print("New value:", newValue)
end)

signal:Update(10)
signal:Update(20)
```
**Output**
``` text
New value: 10
New value: 20
```

---
## Limpieza
Puedes cancelar la suscripción de un `:Listen()` para que deje de ejecutar código cuando se modifica el valor con `:Update()`, lo que también permite ahorrar 
recursos al haber menos callbacks que ejecutar por cada cambio en el valor de un `StateSignal`.

El método `:Listen()` devuelve una `function`, la cual al llamarla cancelas la suscripción del `:Listen()`, lo que ayuda con los recursos una vez el `:Listen()` ya no se necesite más.

``` lua
local disconnect = signal:Listen(function(newValue)
    print(newValue)
end)

signal:Update(50)
disconnect()
signal:Update(100)
```
**Output**
``` text
50
```
Como se puede ver, el 100 no se imprimió, ya que el método `:Listen()` se desconectó antes de ese último `:Update()`.

---
## Otros métodos
### `:Once()`
Ejecuta una sola vez una función (callback) dada (a diferencia de `:Listen()`, no devuelve ninguna función de limpieza, ya que no es necesaria).
``` lua
signal:Once(function(newValue)
    print(newValue)
end)

signal:Update(1)
signal:Update(2)
signal:Update(3)
```
**Output**
``` text
1
```
A diferencia de `:Listen()`, `:Once()` solo se ejecuta una vez.

### `:Run()`
Este método es similar a `:Listen()`, ya que se ejecuta con cada llamada a `:Update()`. Sin embargo, este método ejecuta un callback cuando se llama y cuando se usa `:Update()`
``` lua
local signal = StateSignal.new(0)

signal:Run(function(value)
    print("Value:", value)
end)

signal:Update(100)
```
**Output**
``` text
Value: 0
Value: 100
```
`:Run()` también devuelve una función de limpieza, la cual deja de ejecutar a este método una vez es llamada.
``` lua
local signal = StateSignal.new("Hola")

local disconnect = signal:Run(function(value)
    print("Mensaje:", value)
end)

disconnect()
signal:Update("Adios")
```
**Output**
``` text
Mensaje: Hola
```

### `:Clear()`
Desconecta todos los listeners creados con `:Listen()`, `:Once()` o `:Run()` al instante.
``` lua
signal:Listen(callback)
signal:Once(callback2)
signal:Run(callback3)

signal:Clear() -- Todos los listeners dejan de ejecutarse.
```

---
Lo que hace `StateSignal` para ser mucho más optimizado que usar `ValueBase` es manejar todo por tablas puras, evitando por completo instanciar objetos como `NumberValue`, `StringValue` o incluso `BindableEvent`.
