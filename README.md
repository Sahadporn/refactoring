# refactoring

## Typing game

From Nanthakarn's Typing game <https://github.com/ZEZAY/Penguin-Space>

In the src/view/GameViewManager.java

 <https://github.com/ZEZAY/Penguin-Space/blob/master/src/view/GameViewManager.java>

***Coding Style Error***

- According to java Sun checks it suggest that import .* should be avoided.

```java
import model.*
```

- Typo in method ```dispiayWordLabel()``` should be change to ```displayWordLabel()```

----------
In ```createKeyListeners()```

*I choose to ignore the switch statements.

According to Refactoring Guru: 'When a switch operator performs simple actions, there’s no reason to make code changes.'
Since there are 3 switch statements in total (including the one in ViewManager class) and all of them perform simple actions ex. change boolean attribute value.
It would just make the code more clustered if I refactor all 3 of them with polymorphism or design pattern. 

```java
 private void createKeyListeners() {
        // KeyPressed
        gameScene.setOnKeyPressed(new EventHandler<KeyEvent>() {
            @Override
            public void handle(KeyEvent keyEvent) {
                KeyCode code = keyEvent.getCode();
                switch (code) {
                    case LEFT:
                        isLeftKeyPressed = true;
                        break;
                    case RIGHT:
                        isRightKeyPressed = true;
                        break;
                    case UP:
                        isUpKeyPressed = true;
                        break;
                    case DOWN:
                        isDownKeyPressed = true;
                        break;
                    case ENTER:
                        checkWordLabel();
                        break;
                    case BACK_SPACE:
                        String have = gameWordLabel.getText();
                        gameWordLabel.setText(have.substring(0, have.length() - 1));
                        break;
                    case ESCAPE:
                        gameStage.close();
                        break;
                    default:
                        if (!isUseWordField) dispiayWordLabel();
                        gameWordLabel.setText(gameWordLabel.getText() + code.getChar());
                        break;
                }
            }
        });
        // KeyReleased
        gameScene.setOnKeyReleased(new EventHandler<KeyEvent>() {
            @Override
            public void handle(KeyEvent keyEvent) {
                KeyCode code = keyEvent.getCode();
                switch (code) {
                    case LEFT:
                        isLeftKeyPressed = false;
                        break;
                    case RIGHT:
                        isRightKeyPressed = false;
                        break;
                    case UP:
                        isUpKeyPressed = false;
                        break;
                    case DOWN:
                        isDownKeyPressed = false;
                        break;
                }
            }
        });
```

***Refactoring Sign***

- Inline EventHandler is created on setOnKeyPressed.

***Refactoring***

For code clarity define EventHandlers first, then register the handlers.

```java
    EventHandler<KeyEvent> keyPressHandler =
        new EventHandler<>() {
            @Override
            public void handle(KeyEvent keyEvent) {
                KeyCode code = keyEvent.getCode();
                switch (code) {
                    case LEFT:
                        isLeftKeyPressed = true;
                        break;
                    case RIGHT:
                        isRightKeyPressed = true;
                        break;
                    case UP:
                        isUpKeyPressed = true;
                        break;
                    case DOWN:
                        isDownKeyPressed = true;
                        break;
                    case ENTER:
                        checkWordLabel();
                        break;
                    case BACK_SPACE:
                        String have = gameWordLabel.getText();
                        gameWordLabel.setText(have.substring(0, have.length() - 1));
                        break;
                    case ESCAPE:
                        gameStage.close();
                        break;
                    default:
                        if (!isUseWordField) dispiayWordLabel();
                        gameWordLabel.setText(gameWordLabel.getText() + code.getChar());
                        break;
                }
            }
        };

  EventHandler<KeyEvent> keyReleaseHandler =
            new EventHandler<>() {
                @Override
                public void handle(KeyEvent keyEvent) {
                    KeyCode code = keyEvent.getCode();
                    switch (code) {
                        case LEFT:
                            isLeftKeyPressed = false;
                            break;
                        case RIGHT:
                            isRightKeyPressed = false;
                            break;
                        case UP:
                            isUpKeyPressed = false;
                            break;
                        case DOWN:
                            isDownKeyPressed = false;
                            break;
                    }
                }
            };

    gameScene.setOnKeyPressed(keyPressHandler);
    gameScene.setOnKeyReleased(keyReleasedHandler);
```

----------
In ```checkIfElementsCollide()```

```java
private void checkIfElementsCollide() {
        // check hit star
        if (Double.parseDouble(property.getproperty("game.star_radius")) +
            Double.parseDouble(property.getproperty(
                "game.ship_radius")) > calculateDistance(playerShip, star, 10, 8)) {
            setGameElementNewPosition(star);
            score++;
            gameLabel.setText("POINT: " + score);
        }
        // check hit grey meteor
        for (ImageView greyMeteor : greyMeteors) {
            if (Double.parseDouble(property.getproperty("game.meteor_grey_radius")) +
                Double.parseDouble(property.getproperty("game.ship_radius")) >
                calculateDistance(playerShip, greyMeteor, 18, 15))
                {
                    setGameElementNewPosition(greyMeteor);
                    removePlayerLife();
                }
        }
        // check hit brown meteor
        for (ImageView brownMeteor : brownMeteors) {
            if (Double.parseDouble(property.getproperty("game.meteor_brown_radius")) +
                Double.parseDouble(property.getproperty("game.ship_radius")) >
                calculateDistance(playerShip, brownMeteor, 10, 8))
                {
                    setGameElementNewPosition(brownMeteor);
                    removePlayerLife();
                }
        }
    }
```

***Refactoring sign***

- Change string property to double at run time. Which will perform everytime method is call.

***Refactor***

 Define string properties as a member of the class and convert it to dpuble in the constructor.

```java
public class GameViewManager {

    private final PropertyManager property = PropertyManager.getInstance();

    private double starAndShipRadius;
    private double meteorGreyAndShipRadius;
    private double meteorBrownAndShipRadius;

    // Other private attributes

    public GameViewManager() {
        initializeStage();
        createKeyListeners();
        // generate random positions
        randomPositionGenerators = new Random();
        double shipRadius = Double.parseDouble(property.getproperty("game.ship_radius"));
        starAndShipRadius = Double.parseDouble(property.getproperty(
                            "game.star_radius")) + shipRadius;
        meteorGreyAndShipRadius = Double.parseDouble(property.getproperty(
                                "game.meteor_grey_radius")) + shipRadius;
        meteorBrownAndShipRadius = Double.parseDouble(property.getproperty(
                                "game.meteor_brown_radius")) + shipRadius;
    }
```

```java
    private void checkIfElementsCollide() {
        // check hit star
        if (starAndShipRadius > calculateDistance(playerShip, star, 10, 8)) {
            setGameElementNewPosition(star);
            score++;
            gameLabel.setText("POINT: " + score);
        }
        // check hit grey meteor
        for (ImageView greyMeteor : greyMeteors) {
            if (meteorGreyAndShipRadius > calculateDistance(playerShip, greyMeteor, 18, 15)) {
                setGameElementNewPosition(greyMeteor);
                removePlayerLife();
            }
        }
        // check hit brown meteor
        for (ImageView brownMeteor : brownMeteors) {
            if (meteorBrownAndShipRadius > calculateDistance(playerShip, brownMeteor, 10, 8)) {
                setGameElementNewPosition(brownMeteor);
                removePlayerLife();
            }
        }
    }
```

----------

```java
public class GameViewManager {
    // Private attribute in GameViewManager.
    private ImageView playerShip;

    // Other private attribute

    public GameViewManager() {
        ...

    // Some methods...

    private void createPlayerShip(SHIP chosenShip) {
        playerShip = new ImageView(chosenShip.getUrlShip());
        playerShip.setLayoutX(290);
        playerShip.setLayoutY(Double.parseDouble(property.getproperty("game.height")) - 90);
        gamePane.getChildren().add(playerShip);
    }

    // More methods...

    private void moveShip() {
        if (isLeftKeyPressed) {
            if (angle > -30)
                angle -= 5;
            playerShip.setRotate(angle);
            if (playerShip.getLayoutX() > -20)
                playerShip.setLayoutX(playerShip.getLayoutX() - 3);
        }
        if (isRightKeyPressed) {
            if (angle < 30)
                angle += 5;
            playerShip.setRotate(angle);
            if (playerShip.getLayoutX() < 522)
                playerShip.setLayoutX(playerShip.getLayoutX() + 3);
        }
        if (!isRightKeyPressed && !isLeftKeyPressed) {
            if (angle < 0)
                angle += 5;
            if (angle > 0)
                angle -= 5;
            playerShip.setRotate(angle);
        }
        if (isUpKeyPressed) {
            if (playerShip.getLayoutY() > 0)
                playerShip.setLayoutY(playerShip.getLayoutY() - 3);
        }
        if (isDownKeyPressed) {
            if (playerShip.getLayoutY() < Double.parseDouble(property.getproperty("game.height")) - 90)
                playerShip.setLayoutY(playerShip.getLayoutY() + 3);
        }
    }
```

***Refactoring sign***

playerShip has many behaviors it could be an Object.
Code for playerShip will be clearer and can be use in further implementation for newer Ship type.

***Refactor***

- Create a new class name Ship.

```java
public class Ship extends ImageView {
    private ImageView playerShip;
    private final PropertyManager property = PropertyManager.getInstance();
    private final double VERTICAL_LENGTH = 290;
    private final double HORIZONTAL_LENGTH = Double.parseDouble(property.getproperty("game.height")) - 90;

    public Ship(SHIP chosenShip) {
        playerShip = new ImageView(chosenShip.getUrlShip());
        playerShip.setLayoutX(VERTICAL_LENGTH);
        playerShip.setLayoutY(HORIZONTAL_LENGTH);
    }

    void Rotate(int degree) {
        playerShip.setRotate(degree);
    }

    double getVerticalLength() {
        return playerShip.getLayoutX();
    }

    double getHorizontalLength() {
        return playerShip.getLayoutY();
    }

    void setVerticalLength(int interval) {
        playerShip.setLayoutX(getVerticalLength() + interval);
    }

    void setHorizontalLength(int interval) {
        playerShip.setLayoutY(getHorizontalLength() + interval);
    }
}
```

- In GameViewManager, change the playerShip to Ship object.

```java
public class GameViewManager {
    // Private attribute in GameViewManager.
    private Ship playerShip;

    // Other private attribute

    public GameViewManager() {
        ...
```

- In createPlayerShip() call Ship object instead of creating new ImageView.

```java
private void createPlayerShip(SHIP chosenShip) {
        playerShip = new Ship(chosenShip);
        gamePane.getChildren().add(playerShip);
}
```

- in moveShip() use method from the Ship class instead.

```java
    private void moveShip() {
        if (isLeftKeyPressed) {
            if (angle > -30)
                angle -= 5;
            playerShip.Rotate(angle);
            if (playerShip.getVerticalLength() > -20)
                playerShip.setVerticalLength(-3);
        }
        if (isRightKeyPressed) {
            if (angle < 30)
                angle += 5;
            playerShip.Rotate(angle);
            if (playerShip.getVerticalLength() < 522)
                playerShip.setVerticalLength(3);
        }
        if (!isRightKeyPressed && !isLeftKeyPressed) {
            if (angle < 0)
                angle += 5;
            if (angle > 0)
                angle -= 5;
            playerShip.Rotate(angle);
        }
        if (isUpKeyPressed) {
            if (playerShip.getHorizontalLength() > 0)
                playerShip.setHorizontalLength(-3);
        }
        if (isDownKeyPressed) {
            if (playerShip.getHorizontalLength() < Double.parseDouble(property.getproperty("game.height")) - 90)
                playerShip.setHorizontalLength(3);
        }
    }
```

----------
In src/model/InfoGameLabel and src/model/Infolabel

<https://github.com/ZEZAY/Penguin-Space/blob/master/src/model/InfoGameLabel.java>

<https://github.com/ZEZAY/Penguin-Space/blob/master/src/model/InfoLabel.java>

***Refactoring Sign***

- Have duplicate code.

***Refactor***

- Create a GameLabel class and let InfoGameLabel and Infolabel extends it.

New Gamelabel class.

```java
class GameLabel extends Label {
    private final PropertyManager property = PropertyManager.getInstance();

    public GameLabel(String txt, int width, int height, int fontSize) {
        setPrefWidth(width);
        setPrefHeight(height);
        setText(txt);
        setLabelFont(fontSize);
    }

    /** Set Font. */
    private void setLabelFont(int fontSize) {
        try {
            setFont(Font.loadFont(new FileInputStream(new File(property.getproperty("model.font"))), fontSize));
        } catch (FileNotFoundException e) {
            setFont(Font.font(("Verdana"), fontSize));
        }
    }
}
```

```java
public class InfoGameLabel extends GameLabel {

    private final PropertyManager property = PropertyManager.getInstance();
    /**
     * Label for displaying txt.
     *
     * @param txt to display
     */
    public InfoGameLabel(String txt, int width, int height, int fontSize) {
        super(txt, width, height, fontSize);
        setAlignment(Pos.CENTER_LEFT);
        setPadding(new Insets(10, 10, 10, 10));

        BackgroundImage bg = new BackgroundImage(new Image(property.getproperty("model.label"), 130, 50, false, true),
                BackgroundRepeat.NO_REPEAT, BackgroundRepeat.NO_REPEAT, BackgroundPosition.DEFAULT, null);
        setBackground(new Background(bg));
    }

}

// In GameViewManager class
private void createGameElements(SHIP chosenShip) {
        final int WIDTH = 130;
        final int HEIGHT = 50;
        final int FONT_SIZE = 16;

        // some code...
        gameLabel = new InfoGameLabel("POINTS: ", WIDTH, HEIGHT, FONT_SIZE);
```

```java
public class InfoLabel extends GameLabel {

    private final PropertyManager property = PropertyManager.getInstance();

    /**
     * Label for displaying txt.
     *
     * @param txt to display
     */
    public InfoLabel(String txt, int width, int height, int fontSize) {
        super(txt, width, height, fontSize);
        setAlignment(Pos.CENTER);
        setWrapText(true);

        BackgroundImage bg = new BackgroundImage(new Image(property.getproperty("model.textfield_background"), 380, 49, false, true),
                BackgroundRepeat.NO_REPEAT, BackgroundRepeat.NO_REPEAT, BackgroundPosition.DEFAULT, null);
        setBackground(new Background(bg));
    }
}

// In GameViewManager class
private void createGameWordLabel() {
        final int WIDTH = 380;
        final int HEIGHT = 49;
        final int FONT_SIZE = 23;
        gameWordLabel = new InfoLabel("", WIDTH, HEIGHT, FONT_SIZE);
        // some code...
```
