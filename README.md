import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class BrickBreaker extends JFrame implements ActionListener {
    private static final int WIDTH = 800;
    private static final int HEIGHT = 600;
    private static final int PADDLE_WIDTH = 100;
    private static final int PADDLE_HEIGHT = 20;
    private static final int BALL_DIAMETER = 20;
    private static final int BRICK_WIDTH = 80;
    private static final int BRICK_HEIGHT = 30;
    private static final int PADDLE_SPEED = 10;
    private static final int BALL_SPEED = 5;
    private static final int NUM_ROWS = 5;
    private static final int NUM_COLS = 10;

    private JPanel gamePanel;
    private Timer timer;
    private Paddle paddle;
    private Ball ball;
    private Brick[][] bricks;
    private int score;

    public BrickBreaker() {
        setTitle("Brick Breaker");
        setSize(WIDTH, HEIGHT);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        score = 0;

        gamePanel = new JPanel() {
            @Override
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                draw(g);
            }
        };
        gamePanel.setBackground(Color.BLACK);
        setContentPane(gamePanel);
        gamePanel.setLayout(null); // Using null layout for simplicity

        paddle = new Paddle((WIDTH - PADDLE_WIDTH) / 2, HEIGHT - 100);
        ball = new Ball(WIDTH / 2 - BALL_DIAMETER / 2, HEIGHT / 2 - BALL_DIAMETER / 2);

        bricks = new Brick[NUM_ROWS][NUM_COLS];
        int brickWidthWithGap = BRICK_WIDTH + 2; // gap between bricks
        int brickHeightWithGap = BRICK_HEIGHT + 2;
        for (int i = 0; i < NUM_ROWS; i++) {
            for (int j = 0; j < NUM_COLS; j++) {
                bricks[i][j] = new Brick(j * brickWidthWithGap + 30, i * brickHeightWithGap + 50, BRICK_WIDTH, BRICK_HEIGHT);
            }
        }

        timer = new Timer(20, this);
        timer.start();

        addKeyListener(new KeyAdapter() {
            @Override
            public void keyPressed(KeyEvent e) {
                paddle.keyPressed(e);
            }

            @Override
            public void keyReleased(KeyEvent e) {
                paddle.keyReleased(e);
            }
        });
        setFocusable(true);
        requestFocusInWindow();
    }

    public void actionPerformed(ActionEvent e) {
        update();
        checkCollision();
        repaint();
    }

    private void update() {
        paddle.move();
        ball.move();
    }

    private void checkCollision() {
        // Ball with paddle
        if (ball.getBounds().intersects(paddle.getBounds())) {
            ball.setDY(-BALL_SPEED);
        }

        // Ball with bricks
        for (int i = 0; i < NUM_ROWS; i++) {
            for (int j = 0; j < NUM_COLS; j++) {
                Brick b = bricks[i][j];
                if (b.isVisible() && ball.getBounds().intersects(b.getBounds())) {
                    b.setVisible(false);
                    score += 10;
                    ball.setDY(-ball.getDY());
                }
            }
        }
    }

    private void draw(Graphics g) {
        // Draw paddle
        g.setColor(Color.WHITE);
        g.fillRect(paddle.getX(), paddle.getY(), PADDLE_WIDTH, PADDLE_HEIGHT);

        // Draw ball
        g.setColor(Color.RED);
        g.fillOval((int) ball.getX(), (int) ball.getY(), BALL_DIAMETER, BALL_DIAMETER);

        // Draw bricks
        for (int i = 0; i < NUM_ROWS; i++) {
            for (int j = 0; j < NUM_COLS; j++) {
                Brick b = bricks[i][j];
                if (b.isVisible()) {
                    g.setColor(Color.BLUE);
                    g.fillRect(b.getX(), b.getY(), b.getWidth(), b.getHeight());
                    g.setColor(Color.BLACK);
                    g.drawRect(b.getX(), b.getY(), b.getWidth(), b.getHeight());
                }
            }
        }

        // Draw score
        g.setColor(Color.WHITE);
        g.setFont(new Font("Arial", Font.BOLD, 20));
        g.drawString("Score: " + score, 20, 30);
    }

    private class Paddle {
        private int x, y;
        private int dx;

        public Paddle(int x, int y) {
            this.x = x;
            this.y = y;
            dx = 0;
        }

        public void move() {
            x += dx;
            if (x < 0) {
                x = 0;
            } else if (x + PADDLE_WIDTH > WIDTH) {
                x = WIDTH - PADDLE_WIDTH;
            }
        }

        public void keyPressed(KeyEvent e) {
            int key = e.getKeyCode();
            if (key == KeyEvent.VK_LEFT) {
                dx = -PADDLE_SPEED;
            } else if (key == KeyEvent.VK_RIGHT) {
                dx = PADDLE_SPEED;
            }
        }

        public void keyReleased(KeyEvent e) {
            int key = e.getKeyCode();
            if (key == KeyEvent.VK_LEFT || key == KeyEvent.VK_RIGHT) {
                dx = 0;
            }
        }

        public Rectangle getBounds() {
            return new Rectangle(x, y, PADDLE_WIDTH, PADDLE_HEIGHT);
        }

        public int getX() {
            return x;
        }

        public int getY() {
            return y;
        }
    }

    private class Ball {
        private double x, y;
        private double dx, dy;

        public Ball(double x, double y) {
            this.x = x;
            this.y = y;
            dx = dy = BALL_SPEED;
        }

        public void move() {
            x += dx;
            y += dy;

            // Check boundaries
            if (x < 0 || x > WIDTH - BALL_DIAMETER) {
                dx = -dx;
            }
            if (y < 0 || y > HEIGHT - BALL_DIAMETER) {
                dy = -dy;
            }
        }

        public Rectangle getBounds() {
            return new Rectangle((int) x, (int) y, BALL_DIAMETER, BALL_DIAMETER);
        }

        public double getX() {
            return x;
        }

        public double getY() {
            return y;
        }

        public void setDY(double dy) {
            this.dy = dy;
        }

        public double getDY() {
            return dy;
        }
    }

    private class Brick {
        private int x, y;
        private int width, height;
        private boolean visible;

        public Brick(int x, int y, int width, int height) {
            this.x = x;
            this.y = y;
            this.width = width;
            this.height = height;
            visible = true;
        }

        public Rectangle getBounds() {
            return new Rectangle(x, y, width, height);
        }

        public boolean isVisible() {
            return visible;
        }

        public void setVisible(boolean visible) {
            this.visible = visible;
        }

        public int getX() {
            return x;
        }

        public int getY() {
            return y;
        }

        public int getWidth() {
            return width;
        }

        public int getHeight() {
            return height;
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new BrickBreaker().setVisible(true));
    }
}
