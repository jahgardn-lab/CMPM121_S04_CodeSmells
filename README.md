# CMPM121_S04_CodeSmells

Jade Gardner, Cassidy Croy-Perrett, Trinity Willis

Smells chosen: Data Clumps, Primitive Obsession, Large Class


Data Clumps Examples:
`  

class Crop extends Phaser.GameObjects.Sprite {  

    constructor(scene, x, y, textureList, frame) {  

        super(scene, x, y, textureList[frame], null);  

        this.stages = textureList;  

        this.scene = scene;  

        scene.add.existing(this);
        return this;
    }
    remove() {
        this.destroy();
    }
}`

`  

addCropSprite(u, v) {  

        let newCrop = new Crop(this, u * this.tileWidth + this.tileWidth / 2, 
            v * this.tileHeight + this.tileHeight * 3 / 4, plantTypes[1].growthFrames, 0);  

        let plantScale = (this.tileWidth) / (newCrop.width * 2);  

        newCrop.setScale(plantScale, plantScale);  

        newCrop.setVisible(false);  

        this.cropSprites[[u,v].toString()] = newCrop;  

    }`