# Doom
# ============================================

import random as rnd


# ============================================
class Pos:
    def __init__(self, x, y) -> None:
        self.x = x
        self.y = y


# ------------------------------------
class Figure:
    def __init__(self, pos) -> None:
        self.setPos(pos)

    def setPos(self, pos):
        self.pos = pos

    def getPos(self):
        return self.pos

    def setColor(self, color):
        self.color = color

    def getColor(self):
        return self.color

    def isIn(self, x, y) -> bool:
        return False


class Square(Figure):
    def __init__(self, pos, w, h) -> None:
        super().__init__(pos)
        self.w = w
        self.h = h

    def getWidth(self):
        return self.w

    def getHeight(self):
        return self.h

    def isIn(self, x, y) -> bool:
        _pos = super().getPos()
        if (_pos.x < x) and ((_pos.x + self.w) > x) and (_pos.y < y) and ((_pos.y + self.h) > y):
            return True
        return False


# ----------------------------------
class HitMark:
    def __init__(self, cost) -> None:
        self.setCost(cost)
        self.setState(HitMark.State_Normal)

    def setCost(self, cost):
        self.cost = cost

    def getCost(self):
        return self.cost


class SquareHitMark(Square, HitMark):
    def __init__(self, pos, w, h, cost) -> None:
        super().__init__(pos, w, h)
        HitMark.setCost(self, cost)


# ---------------------------------
# класс внутриигрового сообщения
class GameEvent:
    # пустое событие
    Event_None = 0
    # событие таймера
    Event_Tick = 1
    # событие "выстрела" по цели
    Event_Hit = 2

    def __init__(self, type, data) -> None:
        self.type = type
        self.data = data

    def getType(self):
        return self.type

    def getData(self):
        return self.data


class GameLogic:
    def __init__(self, w, h) -> None:
        self.gameboard_width = w
        self.gameboard_height = h
        self.marks = []
        self.hitMarks = []
        self.score = 0

    def processEvent(self, event):
        if event.type == GameEvent.Event_Tick:
            markRandPos = Pos(rnd.randint(20, self.gameboard_width - 20)
                              , rnd.randint(20, self.gameboard_height - 20))

            markSize = rnd.randint(10, 20)
            markCost = 30 - markSize
            markColor = (rnd.randint(0, 255), rnd.randint(0, 255), rnd.randint(0, 255))
            mark = SquareHitMark(markRandPos, markSize, markSize, markCost)
            mark.setColor(markColor)
            self.addHitMark(mark)

        if event.type == GameEvent.Event_Hit:
            self.hit(event.data)

    def addHitMark(self, mark):
        self.marks.append(mark)

    def hit(self, pos):
        for markIndex in range(len(self.marks)):
            mark = self.marks[markIndex]
            if mark.isIn(pos.x, pos.y):
                self.score += mark.getCost()
                self.marks.pop(markIndex)
                self.hitMarks.append(mark)
                break

    def getBoard(self):
        return self.marks

    def getScore(self):
        return self.score
# наш новый класс, который представляет графику и способ взаимодействия с пользователем
class ConsoleGameGui:
    # инициализация точно такая же, как с pygame, только теперь он нам не нужен
    def __init__(self, w, h, logic) -> None:
        self.main_w = w
        self.main_h = h
        self.logic = logic

    # тут будет цикл обработки сообщений и взаимодействия с пользователем
    def run(self):
        running = True
        while running:
            # сначала посылаем сообщение Event_Tick, чтобы была сгенерирована цель
            self.processEvent(GameEvent(GameEvent.Event_Tick, None))

            # отрисовываем то, что сейчас должно быть на доске
            self.draw()

            # запрашиваем команды у пользователя
            print('------------------')
            print('0. exit')
            print('1. hit target')
            cmd = int(input())

            # значение пустого события по умолчанию
            event = GameEvent(GameEvent.Event_None, None)
            # если команда 0 - идем на выход
            if cmd == 0:
                running = False
                continue
            # если команда 1 запрашиваем координаты
            if cmd == 1:
                x = int(input('input X: '))
                y = int(input('input Y: '))
                event = GameEvent(GameEvent.Event_Hit, Pos(x, y))

            # отрабатываем событие
            self.processEvent(event)

    # в отличие от варианта с pygame тут никакой логики нет, но мы все равно оставили этот метод, чтобы повторять ту же структуру класса
    def processEvent(self, event):
        self.logic.processEvent(event)

    # метод отображения содержимого доски
    def draw(self):
        # выводим текущий счет
        score = self.logic.getScore()
        print('------------------')
        print(f'Your score: {score}')

        # выводим текущие цели
        print('Aims:')
        marks = self.logic.getBoard()
        for index, mark in enumerate(marks):
            print(
                f'aim{index}[x = {mark.getPos().x},y={mark.getPos().y}, w = {mark.getWidth()}, h = {mark.getHeight()}]: {mark.getCost()}')


# --------------------------------------------
if __name__ == "__main__":
    width = 800
    height = 600
    gui = ConsoleGameGui(width, height, GameLogic(width, height))
    gui.run()
