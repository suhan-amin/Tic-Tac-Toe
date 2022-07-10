import pygame

import copy
import random

pygame.init()

sw = 800
sh = 600

board = pygame.image.load('board.jpg')
x_img = pygame.image.load('x.png')
o_img = pygame.image.load('o.png')

win = pygame.display.set_mode((sw, sh))

clock = pygame.time.Clock()

boardVals = [[0,0,0], [0,0,0], [0,0,0]]

piecesOnBoard = []

moveCount = 0

gameOver = False

isTwoPlayer = False

winner=''

class Piece(object):
    def __init__(self, x, y, isX):
        self.x = x
        self.y = y
        self.isX = isX
        self.xTrue = self.x - (self.x % 200)
        self.yTrue = self.y - (self.y % 200)
        if isX:
            self.image = x_img
        else:
            self.image = o_img

    def draw(self, win):

        win.blit(self.image, (self.xTrue, self.yTrue))


def redrawGameWindow():
    win.blit(board, (0,0))
    pygame.draw.rect(win, (210,160,50), [600, 0, 200, 600])
    font = pygame.font.SysFont('times new roman', 40)
    smallfont = pygame.font.SysFont('times new roman', 20)
    if moveCount % 2 == 0:
        turn = 'X'
    else:
        turn = 'O'
    turnText = font.render(turn + "'s Turn", 1, (0, 0, 0))
    if not gameOver:  
        win.blit(turnText, (700 - turnText.get_width()/2, 10))

    gameOverText = font.render(winner, 1, (0, 0, 0))
    if gameOver:
        win.blit(gameOverText, (700 - gameOverText.get_width()/2, 20 + turnText.get_height()))

    pressText = font.render("Press", 1, (0,0,0))
    rule1Text = smallfont.render("1 for Single Player", 1, (0,0,0))
    rule2Text = smallfont.render("2 for Multi Player", 1, (0,0,0))
    rule3Text = smallfont.render("Space to Restart", 1, (0,0,0))
    win.blit(pressText, (700 - pressText.get_width()/2, 340 + gameOverText.get_height()))
    win.blit(rule1Text, (700 - rule1Text.get_width()/2, 400 + gameOverText.get_height()))
    win.blit(rule2Text, (700 - rule2Text.get_width()/2, 440 + gameOverText.get_height()))
    win.blit(rule3Text, (700 - rule3Text.get_width()/2, 480 + gameOverText.get_height()))

    for piece in piecesOnBoard:
        piece.draw(win)


    pygame.display.update()

def isGameOver(boardVals):
    zeroFound = False
    for i in boardVals:
        for j in i:
            if j == 0:
                zeroFound = True
    if not zeroFound:
        return True

    # Horizonal Win
    for i in boardVals:
        if i[0] == i[1] and i[0] == i[2] and i[0] != 0:
            return True

    # Vertical Win
    if boardVals[0][0] == boardVals[1][0] and boardVals[0][0] == boardVals[2][0]:
        if boardVals[0][0] != 0:
            return True
    if boardVals[0][1] == boardVals[1][1] and boardVals[0][1] == boardVals[2][1]:
        if boardVals[0][1] != 0:
            return True
    if boardVals[0][2] == boardVals[1][2] and boardVals[0][2] == boardVals[2][2]:
        if boardVals[0][2] != 0:
            return True

    #Diagonal Check
    if boardVals[0][0] == boardVals[1][1] and boardVals[0][0] == boardVals[2][2]:
        if boardVals[0][0] != 0:
            return True

    if boardVals[0][2] == boardVals[1][1] and boardVals[0][2] == boardVals[2][0]:
        if boardVals[0][2] != 0: # Change boardVals[0][0] to boardVals[0][2]
            return True

    # Add return false
    return False

def computerMove(val):
    # 1 = x
    # -1 = o
    for i in range(3):
        for j in range(3):

            bCopy = copy.deepcopy(boardVals)
            if bCopy[i][j] == 0:
                bCopy[i][j] = val
                if isGameOver(bCopy):
                    piecesOnBoard.append(Piece(j*200, i * 200, False)) if val == -1 else piecesOnBoard.append(Piece(j*200, i*200, True))
                    return bCopy

    opVal = val * -1
    for i in range(3):
        for j in range(3):
            bCopy = copy.deepcopy(boardVals)
            if bCopy[i][j] == 0:
                bCopy[i][j] = opVal
                if isGameOver(bCopy):
                    piecesOnBoard.append(Piece(j*200, i*200, False)) if val == -1 else piecesOnBoard.append(Piece(j*200, i*200, True))
                    bCopy[i][j] = val
                    return bCopy

    move = selRanMove()
    if move != None:
        x, y = move
        bCopy = copy.deepcopy(boardVals)
        bCopy[y][x] = val
        piecesOnBoard.append(Piece(x*200, y*200, False)) if val == -1 else piecesOnBoard.append(Piece(j*200, i*200, True))
        return bCopy
    return boardVals


def selRanMove():
    validMoves = []
    for i in range(3):
        for j in range(3):
            if boardVals[i][j] == 0:
                validMoves.append((j,i))
    if len(validMoves) > 0:
        return random.choice(validMoves)
    else:
        return None

run = True 
while run:
    clock.tick(10)

    if not gameOver:
        mouseX, mouseY = pygame.mouse.get_pos()
        click = pygame.mouse.get_pressed()
        if isTwoPlayer:
            if click != (0,0,0):
                if moveCount % 2 == 0:
                    if boardVals[mouseY//200][mouseX//200] == 0:
                        piecesOnBoard.append(Piece(mouseX, mouseY, True))
                        boardVals[mouseY//200][mouseX//200] = 1
                        moveCount += 1
                else:
                    if boardVals[mouseY // 200][mouseX // 200] == 0:
                        piecesOnBoard.append(Piece(mouseX, mouseY, False))
                        boardVals[mouseY // 200][mouseX // 200] = -1
                        moveCount += 1

            gameOver = isGameOver(boardVals)
            if gameOver:
                if moveCount<9:
                    if moveCount % 2 == 1:
                        winner = 'X Won'
                    else:
                        winner = 'O Won' 
                else:
                    winner = 'Tie Game'
        else:
            if moveCount % 2 == 0:
                if click != (0, 0, 0):
                    if boardVals[mouseY // 200][mouseX // 200] == 0:
                        piecesOnBoard.append(Piece(mouseX, mouseY, True))
                        boardVals[mouseY // 200][mouseX // 200] = 1
                        moveCount += 1
                        gameOver = isGameOver(boardVals)
                        if gameOver:
                            if moveCount<9:
                                if moveCount % 2 == 1:
                                    winner = 'X Won'
                                else:
                                    winner = 'O Won'
                            else:
                                winner = 'Tie Game'
                         # print(boardVals)
            else:
                boardVals = computerMove(-1)
                moveCount += 1
                gameOver = isGameOver(boardVals)
                if gameOver:
                    if moveCount<9:
                        if moveCount % 2 == 1:
                            winner = 'X Won'
                        else:
                            winner = 'O Won'
                    else:
                        winner = 'Tie Game'
                # print(boardVals)


    keys = pygame.key.get_pressed()
    if keys[pygame.K_SPACE]:
        boardVals = [[0,0,0], [0,0,0], [0,0,0]]
        piecesOnBoard.clear()
        moveCount = 0
        gameOver = False
    if keys[pygame.K_1]:
        isTwoPlayer = False
    if keys[pygame.K_2]:
        isTwoPlayer = True


    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            run = False

    redrawGameWindow()

pygame.quit()