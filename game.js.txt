// --- CONFIGURACIÓN ---
const config = {
    type: Phaser.AUTO,
    width: window.innerWidth,
    height: window.innerHeight,
    physics: {
        default: 'arcade',
        arcade: {
            gravity: { y: 800 },
            debug: false
        }
    },
    scene: {
        preload: preload,
        create: create,
        update: update
    }
};

// --- VARIABLES ---
let tralalelo;
let obstacles;
let score = 0;
let scoreText;
let gameOver = false;
let jumpSound;
let gameOverSound;

// Lista de los nombres de tus obstáculos (AHORA CON LAS ROCAS)
const obstacleKeys = ['coral', 'anzuelos', 'anclas', 'redes', 'geiseres', 'llantas', 'medusas', 'rocas'];

// --- INICIALIZACIÓN ---
const game = new Phaser.Game(config);

// --- FUNCIONES DEL JUEGO ---

function preload() {
    // Carga todas tus imágenes
    this.load.image('background', 'assets/fondo.png');
    this.load.image('player', 'assets/tralalelo.png');

    // Carga cada imagen de obstáculo
    obstacleKeys.forEach(key => {
        this.load.image(key, `assets/${key}.png`);
    });

    // Carga tus sonidos
    this.load.audio('jump', 'assets/salto.mp3');
    this.load.audio('gameover', 'assets/gameover.mp3');
}

function create() {
    // Fondo
    const bg = this.add.image(0, 0, 'background').setOrigin(0, 0);
    bg.displayWidth = window.innerWidth;
    bg.displayHeight = window.innerHeight;

    // Personaje
    tralalelo = this.physics.add.sprite(window.innerWidth / 4, window.innerHeight / 2, 'player');
    tralalelo.setScale(0.15); // AJUSTA ESTE VALOR para que Tralalelo tenga el tamaño correcto
    tralalelo.setCollideWorldBounds(true);
    tralalelo.setDepth(1); // Para que Tralalelo siempre esté por delante de los obstáculos

    // Sonidos
    jumpSound = this.sound.add('jump');
    gameOverSound = this.sound.add('gameover');

    // Grupo de obstáculos
    obstacles = this.physics.add.group();

    // Controles
    this.input.on('pointerdown', function () {
        if (!gameOver) {
            tralalelo.setVelocityY(-380);
            jumpSound.play();
        }
    });

    // Puntuación
    scoreText = this.add.text(16, 16, 'Puntos: 0', { fontSize: '32px', fill: '#FFFFFF', stroke: '#000000', strokeThickness: 5 });
    scoreText.setDepth(2); // Para que la puntuación esté por delante de todo

    // Generador de obstáculos
    this.time.addEvent({
        delay: 1800, // Cada 1.8 segundos
        callback: addObstacleRow,
        callbackScope: this,
        loop: true
    });

    // Colisiones
    this.physics.add.collider(tralalelo, obstacles, hitObstacle, null, this);
}

function update() {
    if (gameOver) return;

    // Rotación de Tralalelo para que parezca que cae o sube
    if (tralalelo.body.velocity.y < 0) {
        tralalelo.setAngle(-15);
    } else if (tralalelo.body.velocity.y > 0) {
        tralalelo.setAngle(15);
    } else {
        tralalelo.setAngle(0);
    }
}

// --- FUNCIONES PERSONALIZADAS ---

function addObstacleRow() {
    // Elige un obstáculo al azar de tu lista actualizada
    const randomObstacleKey = Phaser.Math.RND.pick(obstacleKeys);

    // Los obstáculos en zig-zag (como las minas) a veces necesitan una posición inicial diferente.
    // Este código lo ajusta un poco para que se vea mejor.
    let yPosition = window.innerHeight / 2;
    if (randomObstacleKey === 'minas_zigzag_buena') { // Cambia 'minas_zigzag_buena' al nombre real de tu archivo de minas
         yPosition = Phaser.Math.Between(window.innerHeight * 0.3, window.innerHeight * 0.7);
    }
    
    const obstacle = obstacles.create(window.innerWidth + 100, yPosition, randomObstacleKey);
    obstacle.setScale(0.5); // AJUSTA ESTE VALOR si tus obstáculos son muy grandes o pequeños
    obstacle.body.setAllowGravity(false);
    obstacle.setVelocityX(-250);

    // Actualiza la puntuación
    if (!gameOver) {
        score++;
        scoreText.setText('Puntos: ' + score);
    }

    // Limpia los obstáculos que salen de la pantalla
    obstacle.checkWorldBounds = true;
    obstacle.outOfBoundsKill = true;
}

function hitObstacle() {
    if (gameOver) return;
    gameOver = true;

    gameOverSound.play();
    this.physics.pause();
    tralalelo.setTint(0xff0000); // Tralalelo se pone rojo

    // Textos de Game Over
    this.add.text(window.innerWidth / 2, window.innerHeight / 2 - 50, 'GAME OVER', { fontSize: '64px', fill: '#FFFFFF', stroke: '#000000', strokeThickness: 6 }).setOrigin(0.5).setDepth(3);
    this.add.text(window.innerWidth / 2, window.innerHeight / 2 + 50, 'Toca para reiniciar', { fontSize: '32px', fill: '#FFFFFF', stroke: '#000000', strokeThickness: 4 }).setOrigin(0.5).setDepth(3);

    // Reiniciar el juego al tocar la pantalla
    this.input.once('pointerdown', () => {
        gameOver = false;
        score = 0;
        this.scene.restart();
    });
}