display.setStatusBar(display.HiddenStatusBar)

math.randomseed(os.time())

-- VARIABLES
local estadoJuego = "inicio"

local puntuacion = 0
local fallos = 0
local maxFallos = 4

local velocidad = 2
local direccionX = 1
local direccionY = 1

local tiempoLimite = 3000
local tiempoMinimo = 1000
local ultimoToque = 0

-- FONDO
local fondo = display.newImageRect("Fondo.png",
    display.contentWidth,
    display.contentHeight)

fondo.x = display.contentCenterX
fondo.y = display.contentCenterY

-- TEXTO
local marcador = display.newText("Puntos: 0",
    display.contentCenterX,
    display.safeScreenOriginY + 30,
    native.systemFont, 24)

local textoFallos = display.newText("Fallos: 0/4",
    display.contentCenterX,
    display.safeScreenOriginY + 60,
    native.systemFont, 20)

local mensaje = display.newText("Toca la pantalla para comenzar",
    display.contentCenterX,
    display.contentCenterY,
    native.systemFont, 24)

-- CIRCULO
local circulo = display.newImageRect("Circulo.png", 80, 80)
circulo.x = display.contentCenterX
circulo.y = display.contentCenterY
circulo.isVisible = false

------------------------------------------------

local function mover(event)

    if estadoJuego ~= "jugando" then
        return
    end

    circulo.x = circulo.x + velocidad * direccionX
    circulo.y = circulo.y + velocidad * direccionY

    local mitad = circulo.contentWidth / 2

    if circulo.x >= display.contentWidth - mitad or circulo.x <= mitad then
        direccionX = -direccionX
    end

    if circulo.y >= display.contentHeight - mitad or circulo.y <= mitad then
        direccionY = -direccionY
    end

    -- Control de tiempo
    if system.getTimer() - ultimoToque > tiempoLimite then
        fallos = fallos + 1
        textoFallos.text = "Fallos: " .. fallos .. "/4"

        circulo.x = math.random(80, display.contentWidth - 80)
        circulo.y = math.random(80, display.contentHeight - 80)

        ultimoToque = system.getTimer()
    end

    if fallos >= maxFallos then
        estadoJuego = "gameOver"
        mensaje.text = "GAME OVER\nToca para reiniciar"
        mensaje.isVisible = true
        circulo.isVisible = false
    end
end

Runtime:addEventListener("enterFrame", mover)

------------------------------------------------

local function tocarCirculo(event)

    if estadoJuego ~= "jugando" then
        return
    end

    if event.phase == "began" then

        puntuacion = puntuacion + 1
        marcador.text = "Puntos: " .. puntuacion

        circulo.x = math.random(80, display.contentWidth - 80)
        circulo.y = math.random(80, display.contentHeight - 80)

        ultimoToque = system.getTimer()

        -- dificultad progresiva
        if puntuacion % 5 == 0 then
            velocidad = velocidad + 1

            if tiempoLimite > tiempoMinimo then
                tiempoLimite = tiempoLimite - 200
            end
        end
    end
end

circulo:addEventListener("touch", tocarCirculo)

------------------------------------------------

local function tocarPantalla()

    if estadoJuego == "inicio" then
        estadoJuego = "jugando"
        mensaje.isVisible = false
        circulo.isVisible = true

        puntuacion = 0
        fallos = 0
        velocidad = 2
        tiempoLimite = 3000

        marcador.text = "Puntos: 0"
        textoFallos.text = "Fallos: 0/4"

        ultimoToque = system.getTimer()

    elseif estadoJuego == "gameOver" then
        estadoJuego = "jugando"
        mensaje.isVisible = false
        circulo.isVisible = true

        puntuacion = 0
        fallos = 0
        velocidad = 2
        tiempoLimite = 3000

        marcador.text = "Puntos: 0"
        textoFallos.text = "Fallos: 0/4"

        ultimoToque = system.getTimer()
    end
end

Runtime:addEventListener("tap", tocarPantalla)
