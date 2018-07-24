var playerX : int := 212
var playerY : int := 25
var bulletX : array 1 .. 10 of int
var bulletY : array 1 .. 10 of int
var char1 : array char of boolean
var bulletTimer : int := 0
var bulletFired : array 1 .. 10 of boolean
var bulletCounter : int := 0
var pressed : boolean := false
var animationStage : boolean := false
var expAnim : int := 0
var startingExpAnim : array 1 .. 11 of int
var expAnimInUse : array 1 .. 11 of boolean
var expX : array 1 .. 11 of int
var expY : array 1 .. 11 of int
var pickedEnemy : int := 0
var playerDead : boolean := false
var gameOver : boolean := false
var playerLives : int := 2
var font : int
font := Font.New ("emulogic:20")
var font2 := Font.New ("emulogic:40")
var currentLevel : int := 1
var startingLevel : int := 0
var levelStarting : boolean := false
var score : int := 0
var scoreText : string := ""
var totalDead : int := 0
var expAnimOld : int := 0
var soundOn : boolean := true
var stillAlive : boolean := false
var scoreOld : int := 0
var reset : int := 0

var logo : int
logo := Pic.FileNew ("logo.gif")

var playerShip : int
playerShip := Pic.FileNew ("playerShip.gif")
var bullet : int
bullet := Pic.FileNew ("bullet.gif")
var enemy1Apic : int
enemy1Apic := Pic.FileNew ("enemy1a.gif")
var enemy1Bpic : int
enemy1Bpic := Pic.FileNew ("enemy1b.gif")
var background : int
background := Pic.FileNew ("background.gif")

var explosion1 : int
explosion1 := Pic.FileNew ("explosion1.gif")
var explosion2 : int
explosion2 := Pic.FileNew ("explosion2.gif")
var explosion3 : int
explosion3 := Pic.FileNew ("explosion3.gif")
var explosion4 : int
explosion4 := Pic.FileNew ("explosion4.gif")
var explosion5 : int
explosion5 := Pic.FileNew ("explosion5.gif")
var explosion6 : int
explosion6 := Pic.FileNew ("explosion6.gif")

var stars : array 1 .. 6 of int
var star100 : int
star100 := Pic.FileNew ("star100.gif")
var star80 : int
star80 := Pic.FileNew ("star80.gif")
var star60 : int
star60 := Pic.FileNew ("star60.gif")
var star40 : int
star40 := Pic.FileNew ("star40.gif")
var star20 : int
star20 := Pic.FileNew ("star20.gif")
var star0 : int
star0 := Pic.FileNew ("star0.gif")

var level : array 1 .. 6 of int
var level1 : int
level1 := Pic.FileNew ("level1.gif")
var level5 : int
level5 := Pic.FileNew ("level5.gif")
var level10 : int
level10 := Pic.FileNew ("level10.gif")
var level20 : int
level20 := Pic.FileNew ("level20.gif")
var level30 : int
level30 := Pic.FileNew ("level30.gif")
var level40 : int
level40 := Pic.FileNew ("level40.gif")

var levelPos : array 1 .. 6 of int
var levelPosInUse : array 1 .. 6 of boolean

for i : 1 .. 6
    levelPos (i) := 425 + (35 * i)
    levelPosInUse (i) := false
end for

var enemy1 : array 1 .. 21 of int
var enemyDead : array 1 .. 21 of boolean
var enemyX : array 1 .. 21 of int
var enemyY : array 1 .. 21 of int
var enemy1a : array 1 .. 21 of int
var enemy1b : array 1 .. 21 of int
var explosion : array 1 .. 6 of int
var enemyLaunched : array 1 .. 21 of boolean
var starX : array 1 .. 50 of int
var starY : array 1 .. 50 of int
var starVisible : array 1 .. 50 of int
var starMinus : array 1 .. 50 of boolean

View.Set ("graphics: 600;500, offscreenonly")
Pic.Draw (background, 0, 0, picCopy)
for i : 1 .. 10
    bulletX (i) := -100
    bulletY (i) := -100
    bulletFired (i) := false
end for
for i : 1 .. 21

    enemy1a (i) := enemy1Apic
    enemy1b (i) := enemy1Bpic
    enemyDead (i) := false
    enemyLaunched (i) := false
    if i < 8 then
	enemyX (i) := 25 + (45 * i)
	enemyY (i) := maxy - (50 * 1)
    elsif i < 15 then
	enemyX (i) := 25 + (45 * (i - 7))
	enemyY (i) := maxy - (50 * 2)
    else
	enemyX (i) := 25 + (45 * (i - 14))
	enemyY (i) := maxy - (50 * 3)
    end if
end for

explosion (1) := explosion1
explosion (2) := explosion2
explosion (3) := explosion3
explosion (4) := explosion4
explosion (5) := explosion5
explosion (6) := explosion6

level (1) := level1
level (2) := level5
level (3) := level10
level (4) := level20
level (5) := level30
level (6) := level40

for i : 1 .. 11
    startingExpAnim (i) := 0
    expX (i) := -100
    expY (i) := -100
    expAnimInUse (i) := false
end for

for i : 1 .. 50
    starX (i) := Rand.Int (0, 450)
    starY (i) := Rand.Int (0, 750)
    starVisible (i) := Rand.Int (0, 10)
    if Rand.Int (1, 2) = 1 then
	starMinus (i) := true
    else
	starMinus (i) := false
    end if
end for
stars (1) := star100
stars (2) := star80
stars (3) := star60
stars (4) := star40
stars (5) := star20
stars (6) := star0

process firingSound
    Music.PlayFile ("sfxFiring.wav")
end firingSound

process explosionSound
    Music.PlayFile ("sfxExplosion.wav")
end explosionSound

procedure updateDisplay
    %hit boxes start%
    for i : 1 .. 10
	drawfillbox (bulletX (i), bulletY (i), bulletX (i) + Pic.Width (bullet) - 1, bulletY (i) + Pic.Height (bullet) - 1, 25)
    end for
    drawfillbox (playerX, playerY, playerX + Pic.Width (playerShip) - 1, playerY + Pic.Height (playerShip) - 1, 35)
    for i : 1 .. 21
	if enemyDead (i) = false then
	    drawfillbox (enemyX (i), enemyY (i), enemyX (i) + Pic.Width (enemy1Apic) - 1, enemyY (i) + Pic.Height (enemy1Apic) - 1, i + 100)
	end if
    end for

    %hit boxes end%

    for i : 1 .. 10
	for j : 1 .. 21
	    if whatdotcolour (bulletX (i), bulletY (i) + Pic.Height (bullet) - 1) = j + 100 or whatdotcolour (bulletX (i) + Pic.Width (bullet) - 1, bulletY (i) + Pic.Height (bullet) - 1) = j + 100
		    then
		score += 100
		enemyDead (j) := true
		enemyLaunched (j) := false
		totalDead += 1
		bulletY (i) := maxy + 11
		if soundOn = true then
		    fork explosionSound
		end if
		for k : 1 .. 10
		    if expAnimInUse (k) = false then
			expAnimInUse (k) := true
			startingExpAnim (k) := expAnim
			expX (k) := enemyX (j) - 19
			expY (k) := enemyY (j) - 22
			Pic.Draw (explosion ((expAnim - startingExpAnim (k)) + 1), expX (k), expY (k), picMerge)
			exit
		    end if
		end for
	    end if
	end for
    end for

    for i : 1 .. 21
	if whatdotcolour (playerX, playerY) = i + 100 and playerDead = false then
	    playerDead := true
	    expX (11) := playerX - 17
	    expY (11) := playerY - 16
	    expAnimInUse (11) := true
	    startingExpAnim (11) := expAnim
	    Pic.Draw (explosion ((expAnim - startingExpAnim (11)) + 4), expX (11), expY (11), picMerge)
	    playerX := -200
	    playerY := -200
	    enemyDead (i) := true
	    enemyLaunched (i) := false
	    totalDead += 1
	    for k : 1 .. 10
		if expAnimInUse (k) = false then
		    expAnimInUse (k) := true
		    startingExpAnim (k) := expAnim
		    expX (k) := enemyX (i) - 19
		    expY (k) := enemyY (i) - 22
		    Pic.Draw (explosion ((expAnim - startingExpAnim (k)) + 1), expX (k), expY (k), picMerge)
		    exit
		end if
	    end for
	    exit
	elsif whatdotcolour (playerX, playerY + Pic.Height (playerShip) - 1) = i + 100 and playerDead = false then
	    playerDead := true
	    expX (11) := playerX - 17
	    expY (11) := playerY - 16
	    expAnimInUse (11) := true
	    startingExpAnim (11) := expAnim
	    Pic.Draw (explosion ((expAnim - startingExpAnim (11)) + 4), expX (11), expY (11), picMerge)
	    playerX := -200
	    playerY := -200
	    enemyDead (i) := true
	    enemyLaunched (i) := false
	    totalDead += 1
	    for k : 1 .. 10
		if expAnimInUse (k) = false then
		    expAnimInUse (k) := true
		    startingExpAnim (k) := expAnim
		    expX (k) := enemyX (i) - 19
		    expY (k) := enemyY (i) - 22
		    Pic.Draw (explosion ((expAnim - startingExpAnim (k)) + 1), expX (k), expY (k), picMerge)
		    exit
		end if
	    end for
	    exit
	elsif whatdotcolour (playerX + Pic.Width (playerShip) - 1, playerY) = i + 100 and playerDead = false then
	    playerDead := true
	    expX (11) := playerX - 17
	    expY (11) := playerY - 16
	    expAnimInUse (11) := true
	    startingExpAnim (11) := expAnim
	    Pic.Draw (explosion ((expAnim - startingExpAnim (11)) + 4), expX (11), expY (11), picMerge)
	    playerX := -200
	    playerY := -200
	    enemyDead (i) := true
	    enemyLaunched (i) := false
	    totalDead += 1
	    for k : 1 .. 10
		if expAnimInUse (k) = false then
		    expAnimInUse (k) := true
		    startingExpAnim (k) := expAnim
		    expX (k) := enemyX (i) - 19
		    expY (k) := enemyY (i) - 22
		    Pic.Draw (explosion ((expAnim - startingExpAnim (k)) + 1), expX (k), expY (k), picMerge)
		    exit
		end if
	    end for
	    exit
	elsif whatdotcolour (playerX + Pic.Width (playerShip) - 1, playerY + Pic.Height (playerShip) - 1) = i + 100 and playerDead = false then
	    playerDead := true
	    expX (11) := playerX - 17
	    expY (11) := playerY - 16
	    expAnimInUse (11) := true
	    startingExpAnim (11) := expAnim
	    Pic.Draw (explosion ((expAnim - startingExpAnim (11)) + 4), expX (11), expY (11), picMerge)
	    playerX := -200
	    playerY := -200
	    enemyDead (i) := true
	    enemyLaunched (i) := false
	    totalDead += 1
	    for k : 1 .. 10
		if expAnimInUse (k) = false then
		    expAnimInUse (k) := true
		    startingExpAnim (k) := expAnim
		    expX (k) := enemyX (i) - 19
		    expY (k) := enemyY (i) - 22
		    Pic.Draw (explosion ((expAnim - startingExpAnim (k)) + 1), expX (k), expY (k), picMerge)
		    exit
		end if
	    end for
	    exit
	end if
    end for

    % %picture start%
    Pic.Draw (background, 0, 0, picMerge)

    for i : 1 .. 50
	if bulletTimer > 10 * i and starVisible (i) = 4 then
	    Pic.Draw (star20, starX (i), starY (i), picMerge)
	elsif bulletTimer > 10 * i and starVisible (i) = 5 then
	    Pic.Draw (star40, starX (i), starY (i), picMerge)
	elsif bulletTimer > 10 * i and starVisible (i) = 6 then
	    Pic.Draw (star60, starX (i), starY (i), picMerge)
	elsif bulletTimer > 10 * i and starVisible (i) = 7 then
	    Pic.Draw (star80, starX (i), starY (i), picMerge)
	elsif bulletTimer > 10 * i and starVisible (i) = 8 then
	    Pic.Draw (star100, starX (i), starY (i), picMerge)
	elsif bulletTimer > 10 * i and starVisible (i) = 9 then
	    Pic.Draw (star100, starX (i), starY (i), picMerge)
	elsif bulletTimer > 10 * i and starVisible (i) = 10 then
	    Pic.Draw (star100, starX (i), starY (i), picMerge)
	end if
    end for

    for i : 1 .. 21
	if animationStage = true and enemyDead (i) = false then
	    Pic.Draw (enemy1a (i), enemyX (i), enemyY (i), picMerge)
	elsif animationStage = false and enemyDead (i) = false then
	    Pic.Draw (enemy1b (i), enemyX (i), enemyY (i), picMerge)
	end if
    end for
    for i : 1 .. 11
	if ((expAnim - startingExpAnim (i)) + 1) <= 3 and expAnimInUse (i) = true then
	    if i <= 10 then
		Pic.Draw (explosion ((expAnim - startingExpAnim (i)) + 1), expX (i), expY (i), picMerge)
	    else
		Pic.Draw (explosion ((expAnim - startingExpAnim (i)) + 4), expX (i), expY (i), picMerge)
	    end if
	else
	    expX (i) := -100
	    expY (i) := -100
	    expAnimInUse (i) := false
	end if
    end for
    for i : 1 .. 10
	Pic.Draw (bullet, bulletX (i), bulletY (i), picMerge)
    end for
    %picture end%
    Pic.Draw (playerShip, playerX, playerY, picMerge)
    for i : 0 .. playerLives - 1
	Pic.Draw (playerShip, 460 + (35 * i), 350, picMerge)
    end for

    startingLevel := currentLevel
    for i : 1 .. 10
	if startingLevel - 10 >= 0 then
	    startingLevel -= 10
	    Pic.Draw (level (3), 440 + (20 * i), 200, picMerge)
	elsif startingLevel - 5 >= 0 then
	    startingLevel -= 5
	    Pic.Draw (level (2), 440 + (20 * i), 200, picMerge)
	elsif startingLevel - 1 >= 0 then
	    startingLevel -= 1
	    Pic.Draw (level (1), 440 + (20 * i), 200, picMerge)
	end if
	if startingLevel = 0 then
	    exit
	end if
    end for
    Draw.Text ("Score:", 460, 425, font, white)
    Draw.Text (intstr (score), 460, 400, font, white)
    Time.Delay (16)
    View.Update
end updateDisplay

procedure launchEnemy

    loop
	pickedEnemy := Rand.Int (1, 21)
	if enemyLaunched (pickedEnemy) = false and enemyDead (pickedEnemy) = false then
	    enemyLaunched (pickedEnemy) := true
	    enemyY (pickedEnemy) -= 1
	    exit
	end if
	if totalDead >= 21 then
	    totalDead := 0
	    currentLevel += 1
	    expAnimOld := expAnim
	    loop
		bulletTimer += 1
		if bulletTimer mod 10 = 0 then
		    expAnim += 1
		    updateDisplay
		    Time.Delay (200)
		    exit when expAnim >= expAnimOld + 3
		end if
	    end loop
	    for r : 1 .. 21
		enemyDead (r) := false
		enemyLaunched (r) := false
		if r < 8 then
		    enemyX (r) := 25 + (45 * r)
		    enemyY (r) := maxy
		elsif r < 15 then
		    enemyX (r) := 25 + (45 * (r - 7))
		    enemyY (r) := maxy
		else
		    enemyX (r) := 25 + (45 * (r - 14))
		    enemyY (r) := maxy
		end if
	    end for
	    for i : 1 .. 10
		bulletX (i) := -100
		bulletY (i) := -100
		bulletFired (i) := false
	    end for
	    updateDisplay
	    View.Update
	    Draw.Text ("READY?", 250, 375, font, white)
	    View.Update
	    Time.Delay (2000)
	    bulletTimer := 0
	    exit
	elsif totalDead < 21 then
	    for i : 1 .. 21
		if enemyLaunched (i) = true then
		    stillAlive := true
		    exit
		end if
	    end for
	    if stillAlive = true then
		stillAlive := false
		exit
	    end if
	end if
    end loop
end launchEnemy

loop
    bulletTimer += 1
    Pic.Draw (background, 0, 0, picMerge)
    Pic.Draw (logo, ((maxx div 2) - ((Pic.Width (logo) div 2) - 1)), ((maxy div 2) - ((Pic.Height (logo) div 2) - 1)) + 200, picMerge)
    if bulletTimer mod 100 = 0 then
	if animationStage = true then
	    animationStage := false
	else
	    animationStage := true
	end if
    end if
    if animationStage = true then
	Draw.Text ("Press enter to play with sounds", (maxx div 2) - 185, maxy div 2, font, white)
	Draw.Text ("Press shift to play without sounds", (maxx div 2) - 200, (maxy div 2) - 50, font, white)
    end if
    Draw.Text ("Use the left and right arrow keys to move", (maxx div 2) - 250, (maxy div 2) - 150, font, white)
    Draw.Text ("Use the F key to fire", (maxx div 2) - 125, (maxy div 2) - 180, font, white)
    Draw.Text ("You gain points over time for being alive,", (maxx div 2) - 245, (maxy div 2) - 210, font, white)
    Draw.Text ("you also get 100 points for each enemy you hit", (maxx div 2) - 285, (maxy div 2) - 240, font, white)
    Draw.Text ("Your goal is to kill as many enemies as you can,", (maxx div 2) - 290, (maxy div 2) - 270, font, white)
    Draw.Text ("they get faster with each level", (maxx div 2) - 175, (maxy div 2) - 300, font, white)
    Draw.Text ("You get an extra life every 10 000 points", (maxx div 2) - 245, (maxy div 2) - 330, font, white)
    View.Update
    Input.KeyDown (char1)
    if char1 (KEY_ENTER) then
	cls
	exit
    elsif char1 (KEY_SHIFT) then
	soundOn := false
	cls
	exit
    end if
end loop

Pic.Draw (background, 0, 0, picMerge)
Draw.Text ("READY?", 250, 375, font, white)
for i : 0 .. playerLives - 1
    Pic.Draw (playerShip, 460 + (35 * i), 350, picMerge)
end for
startingLevel := currentLevel
for i : 1 .. 10
    if startingLevel - 10 >= 0 then
	startingLevel -= 10
	Pic.Draw (level (3), 440 + (20 * i), 200, picMerge)
    elsif startingLevel - 5 >= 0 then
	startingLevel -= 5
	Pic.Draw (level (2), 440 + (20 * i), 200, picMerge)
    elsif startingLevel - 1 >= 0 then
	startingLevel -= 1
	Pic.Draw (level (1), 440 + (20 * i), 200, picMerge)
    end if
    if startingLevel = 0 then
	exit
    end if
end for
View.Update
Time.Delay (2000)

loop
    bulletTimer += 1
    for i : 1 .. 10
	if bulletY (i) < maxy + 10 and bulletFired (i) = true then
	    bulletY (i) += 10
	end if
	if bulletY (i) > maxy + 10 then
	    bulletX (i) := -100
	    bulletY (i) := -100
	    bulletFired (i) := false
	end if
    end for

    if score >= scoreOld + 10000 and playerLives <= 4 then
	scoreOld := score
	playerLives += 1
    end if

    if playerDead = true then
	playerLives -= 1
	for i : 1 .. 10
	    bulletX (i) := -100
	    bulletY (i) := -100
	    bulletFired (i) := false
	end for
	if playerLives <= 0 then
	    gameOver := true
	else
	    loop
		bulletTimer += 1
		if bulletTimer mod 10 = 0 then
		    expAnim += 1
		    updateDisplay
		    Time.Delay (200)
		end if
		if bulletTimer mod 5 = 0 then
		    for i : 1 .. 50
			if starMinus (i) = false then
			    starVisible (i) += 1
			    if starVisible (i) = 10 then
				starMinus (i) := true
			    end if
			else
			    starVisible (i) -= 1
			    if starVisible (i) = 0 then
				starMinus (i) := false
			    end if
			end if
		    end for
		end if
		if bulletTimer mod 17 = 0 then
		    pressed := false
		    if animationStage = true then
			animationStage := false
		    else
			animationStage := true
		    end if
		end if
		exit when expAnim = startingExpAnim (11) + 3
	    end loop
	    loop
		bulletTimer += 1
		for i : 1 .. 21
		    if enemyLaunched (i) = true then
			enemyY (i) -= 3 + currentLevel
			if enemyY (i) < -20 then
			    enemyLaunched (i) := false
			    if i < 8 then
				enemyX (i) := 25 + (45 * i)
				enemyY (i) := maxy
			    elsif i < 15 then
				enemyX (i) := 25 + (45 * (i - 7))
				enemyY (i) := maxy
			    else
				enemyX (i) := 25 + (45 * (i - 14))
				enemyY (i) := maxy
			    end if
			end if
		    end if
		end for
		for i : 1 .. 21
		    if enemyLaunched (i) = false then
			reset += 1
		    else
			reset := 0
			exit
		    end if
		end for
		if bulletTimer mod 5 = 0 then
		    for i : 1 .. 50
			if starMinus (i) = false then
			    starVisible (i) += 1
			    if starVisible (i) = 10 then
				starMinus (i) := true
			    end if
			else
			    starVisible (i) -= 1
			    if starVisible (i) = 0 then
				starMinus (i) := false
			    end if
			end if
		    end for
		end if
		if bulletTimer mod 17 = 0 then
		    pressed := false
		    if animationStage = true then
			animationStage := false
		    else
			animationStage := true
		    end if
		end if
		exit when reset = 21
		updateDisplay
	    end loop
	    loop
		bulletTimer += 1
		for i : 1 .. 21
		    if i < 8 then
			if enemyY (i) not= maxy - (50 * 1) and enemyLaunched (i) = false then
			    enemyY (i) -= 2
			end if
			if enemyY (i) = maxy - (50 * 1) or enemyDead (i) = true then
			    reset += 1
			end if
		    elsif i < 15 then
			if enemyY (i) not= maxy - (50 * 2) and enemyLaunched (i) = false then
			    enemyY (i) -= 2
			end if
			if enemyY (i) = maxy - (50 * 2) or enemyDead (i) = true then
			    reset += 1
			end if
		    else
			if enemyY (i) not= maxy - (50 * 3) and enemyLaunched (i) = false then
			    enemyY (i) -= 2
			end if
			if enemyY (i) = maxy - (50 * 3) or enemyDead (i) = true then
			    reset += 1
			end if
		    end if
		end for
		if bulletTimer mod 5 = 0 then
		    for i : 1 .. 50
			if starMinus (i) = false then
			    starVisible (i) += 1
			    if starVisible (i) = 10 then
				starMinus (i) := true
			    end if
			else
			    starVisible (i) -= 1
			    if starVisible (i) = 0 then
				starMinus (i) := false
			    end if
			end if
		    end for
		end if
		if bulletTimer mod 17 = 0 then
		    pressed := false
		    if animationStage = true then
			animationStage := false
		    else
			animationStage := true
		    end if
		end if
		if reset not= 21 then
		    reset := 0
		end if
		exit when reset = 21
		updateDisplay
	    end loop
	    reset := 0
	    View.Update
	    Draw.Text ("READY?", 250, 375, font, white)
	    View.Update
	    Time.Delay (2000)
	    playerX := 212
	    playerY := 25
	    playerDead := false
	end if
    end if

    if gameOver = true then
	Pic.Draw (background, 0, 0, picMerge)
	Draw.Text ("Game Over", 160, 475, font2, white)
	scoreText := "Score: " + intstr (score)
	Draw.Text (scoreText, 200 - (length (intstr (score)) * 13), 400, font2, white)

	View.Update
	exit
    end if

    for i : 1 .. 50
	if bulletTimer > 10 * i then
	    starY (i) -= 7
	    if starY (i) < -10 then
		starY (i) := maxy + 10
	    end if
	end if
    end for

    for i : 1 .. 21
	if enemyLaunched (i) = true and playerDead = false then
	    if enemyX (i) <= playerX then
		enemyX (i) += ((Math.Distance (enemyX (i), 0, playerX, 0)) div 25)
	    else
		enemyX (i) += (((Math.Distance (enemyX (i), 0, playerX, 0)) div 25) * -1)
	    end if
	    enemyY (i) -= 3 + currentLevel
	    if enemyY (i) < -20 then
		enemyLaunched (i) := false
		if i < 8 then
		    enemyX (i) := 25 + (45 * i)
		    enemyY (i) := maxy
		elsif i < 15 then
		    enemyX (i) := 25 + (45 * (i - 7))
		    enemyY (i) := maxy
		else
		    enemyX (i) := 25 + (45 * (i - 14))
		    enemyY (i) := maxy
		end if
	    end if
	end if
    end for

    for i : 1 .. 21
	if i < 8 then
	    if enemyY (i) not= maxy - (50 * 1) and enemyLaunched (i) = false then
		enemyY (i) -= 2
	    end if
	elsif i < 15 then
	    if enemyY (i) not= maxy - (50 * 2) and enemyLaunched (i) = false then
		enemyY (i) -= 2
	    end if
	else
	    if enemyY (i) not= maxy - (50 * 3) and enemyLaunched (i) = false then
		enemyY (i) -= 2
	    end if
	end if
    end for

    Input.KeyDown (char1)
    if char1 ('f') and pressed = false then
	if bulletCounter < 10 then
	    bulletCounter += 1
	    bulletX (bulletCounter) := playerX + 12
	    bulletY (bulletCounter) := playerY + 30
	    bulletFired (bulletCounter) := true
	    pressed := true
	else
	    bulletCounter := 0
	end if
	if soundOn = true then
	    fork firingSound
	end if
    end if
    if char1 (KEY_RIGHT_ARROW) and playerX <= 370 then
	playerX += 5
    elsif char1 (KEY_LEFT_ARROW) and playerX >= 50 then
	playerX -= 5
    end if
    if bulletTimer mod 17 = 0 then
	pressed := false
	if animationStage = true then
	    animationStage := false
	else
	    animationStage := true
	end if
    end if
    if bulletTimer mod 10 = 0 then
	expAnim += 1
	score += 1
    end if
    if bulletTimer mod 30 = 0 then
	score += 1
    end if
    if bulletTimer mod 100 - currentLevel = 0 then
	launchEnemy
    end if
    if bulletTimer mod 75 - currentLevel = 0 and currentLevel >= 5 then
	launchEnemy
    end if
    if bulletTimer mod 60 - currentLevel = 0 and currentLevel >= 10 then
	launchEnemy
    end if
    if bulletTimer mod 5 = 0 then
	for i : 1 .. 50
	    if starMinus (i) = false then
		starVisible (i) += 1
		if starVisible (i) = 10 then
		    starMinus (i) := true
		end if
	    else
		starVisible (i) -= 1
		if starVisible (i) = 0 then
		    starMinus (i) := false
		end if
	    end if
	end for
    end if
    updateDisplay
end loop
