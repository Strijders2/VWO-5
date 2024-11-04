var kolommenMetBom = [];
var spelActief = true;
var levens = 2; // Begin met 2 levens

class Raster {
  constructor(r,k) {
    this.aantalRijen = r;
    this.aantalKolommen = k;
    this.celGrootte = null;
  }

  berekenCelGrootte() {
    this.celGrootte = canvas.width / this.aantalKolommen;
  }

  teken() {
    push();
    noFill();
    stroke('grey');
    for (var rij = 0;rij < this.aantalRijen;rij++) {
      for (var kolom = 0;kolom < this.aantalKolommen;kolom++) {
        rect(kolom*this.celGrootte,rij*this.celGrootte,this.celGrootte,this.celGrootte);
      }
    }
    pop();
  }
}

class Jos {
  constructor() {
    this.x = 400;
    this.y = 300;
    this.animatie = [];
    this.frameNummer =  3;
    this.stapGrootte = null;
    this.gehaald = false;
  }

  beweeg() {
    if (keyIsDown(65)) {
      this.x -= this.stapGrootte;
      this.frameNummer = 2;
    }
    if (keyIsDown(68)) {
      this.x += this.stapGrootte;
      this.frameNummer = 1;
    }
    if (keyIsDown(87)) {
      this.y -= this.stapGrootte;
      this.frameNummer = 4;
    }
    if (keyIsDown(83)) {
      this.y += this.stapGrootte;
      this.frameNummer = 5;
    }

    this.x = constrain(this.x,0,canvas.width);
    this.y = constrain(this.y,0,canvas.height - raster.celGrootte);

    if (this.x == canvas.width) {
      this.gehaald = true;
    }
  }

  wordtGeraakt(vijand) {
    if (this.x == vijand.x && this.y == vijand.y) {
      return true;
    }
    else {
      return false;
    }
  }
  wordtGeraakt(object) {
    // Check for collision with any object that has a `width` and `height` property
    if (this.x + raster.celGrootte >= object.x &&
        this.x <= object.x + object.width &&
        this.y + raster.celGrootte >= object.y &&
        this.y <= object.y + object.height) {
      return true; // Collision detected
    } else {
      return false;
    }
  }
  toon() {
    image(this.animatie[this.frameNummer],this.x,this.y,raster.celGrootte,raster.celGrootte);
  }
}  

class Vijand {
  constructor(x,y) {
    this.x = x;
    this.y = y;
    this.sprite = null;
    this.stapGrootte = null;
  }

  beweeg() {
    this.x += floor(random(-1,2))*this.stapGrootte;
    this.y += floor(random(-1,2))*this.stapGrootte;

    this.x = constrain(this.x,0,canvas.width - raster.celGrootte);
    this.y = constrain(this.y,0,canvas.height - raster.celGrootte);
  }

  toon() {
    image(this.sprite,this.x,this.y,raster.celGrootte,raster.celGrootte);
  }
}
class Bom {
  constructor(x, y) {
    this.x = x; // De x-positie van de bom, uitgelijnd op raster
    this.y = y; // De y-positie van de bom, uitgelijnd op raster
    this.sprite = loadImage("images/sprites/bom.png"); // Laadt bom-afbeelding
    this.stapGrootte = null; // Stapgrootte wordt later ingesteld
    this.beweegRichting = 1; // Richting waarin de bom beweegt
    this.speed = random(0.3, 0.8); // Willekeruige snelheid voor elke bom, adjusted for slower movement
    
    // Initialize the bom's movement direction based on its initial position
    if (this.y === 0) {
      this.beweegRichting = 1; // Start moving down
    } else if (this.y === canvas.height - raster.celGrootte) {
      this.beweegRichting = -1; // Start moving up
    }
  }

  beweeg() {
    // Beweeg de bom op en neer binnen het raster
    this.y += this.beweegRichting * this.speed * this.stapGrootte; // Hier wordt "speed"          gebruikt

    // Verander de richting als de bom de rand van het raster bereikt
    if (this.y <= 0 || this.y >= canvas.height - raster.celGrootte) {
      this.beweegRichting *= -1; // Verander richting
    }
  }

  toon() {
    image(this.sprite, this.x, this.y, raster.celGrootte, raster.celGrootte); // Toon de bom binnen het raster
  }
}
class Appel {
  constructor() {
    this.x = floor(random(1, raster.aantalKolommen)) * raster.celGrootte + 5;
    this.y = floor(random(0, raster.aantalRijen)) * raster.celGrootte + 4;
    this.sprite = loadImage("images/sprites/appel_2.png");
  }//de appel wordt perfect in een hokje geplaatst

  toon() {
    image(this.sprite, this.x, this.y, raster.celGrootte - 10, raster.celGrootte - 10);
  }
}

function preload() {
  brug = loadImage("images/backgrounds/dame_op_brug_1800.jpg");
}
function setup() {
  canvas = createCanvas(900, 600);
  canvas.parent();
  frameRate(10);
  textFont("Verdana");
  textSize(90);

  raster = new Raster(12, 18);  // Raster van 12 rijen en 18 kolommen
  raster.berekenCelGrootte();   // Bereken de grootte van elke cel

  eve = new Jos();  // Speler
  eve.stapGrootte = raster.celGrootte;
  for (var b = 0; b < 6; b++) {
    frameEve = loadImage("images/sprites/Eve100px/Eve_" + b + ".png");
    eve.animatie.push(frameEve);
  }

  alice = new Vijand(700, 200);
  alice.stapGrootte = eve.stapGrootte;
  alice.sprite = loadImage("images/sprites/Alice100px/Alice.png");

  bob = new Vijand(600, 400);
  bob.stapGrootte = eve.stapGrootte;
  bob.sprite = loadImage("images/sprites/Bob100px/Bob.png");

  appel = new Appel();
  appel.sprite = loadImage("images/sprites/appel_2.png");
          // Plaatje van de groene appel. 
  // Voeg vijf bommen toe, uitgelijnd op het raster
  bommen = [];
  for (let i = 0; i < 5; i++) {
    // Willekeurige startpositie binnen het raster
    let startX = floor(random(0, raster.aantalKolommen)) * raster.celGrootte; // Veelvoud van raster.celGrootte
    let startY = floor(random(0, raster.aantalRijen)) * raster.celGrootte;    // Veelvoud van raster.celGrootte
    let bom = new Bom(startX, startY);  // Geef startX en startY door aan de constructor van Bom
    bom.stapGrootte = raster.celGrootte; // Stapgrootte is gelijk aan celgrootte
    bommen.push(bom);  // Voeg de bom toe aan de lijst van bommen
  }
}
function draw() {
  background(brug);  // Achtergrondafbeelding
  raster.teken();    // Raster tekenen
               
  fill('Black');  // Toon levens linksbovenin
  textSize(24);
  text("Levens: " + levens, 20, 30); // Positie linksbovenin
  eve.beweeg();      // Beweging van speler
  alice.beweeg();    // Beweging van vijand Alice
  bob.beweeg();      // Beweging van vijand Bob

  eve.toon();        // Toon speler
  alice.toon();      // Toon Alice
  bob.toon();        // Toon Bob
  appel.toon();      // Toon de groene appel

  // Toon en beweeg de vijf bommen
  for (let bom of bommen) {
    bom.beweeg();    // Beweeg elke bom op en neer
    bom.toon();      // Toon elke bom

    // Check of de speler geraakt wordt door een van de bommen
    if (eve.wordtGeraakt(bom)) {
      noLoop();      // Stop het spel als de speler geraakt wordt
      return;
    }
  }

  // Check of de speler geraakt wordt door Alice of Bob
  if (eve.wordtGeraakt(alice) || eve.wordtGeraakt(bob)) {
    noLoop();        // Stop het spel als de speler geraakt wordt
  }
  if (levens <= 0) {
      noLoop(); // Stop het spel als de levens op zijn
      background('red');
      fill('white');
      textSize(50);
      text("Je hebt verloren", 300, 300);
  }

  // Check of de speler het einde heeft bereikt
  if (eve.gehaald) {
    background('green');
    fill('white');
    text("Je hebt gewonnen!", 30, 300);
    noLoop();   // Stop het spel als de speler wint
  }
}
