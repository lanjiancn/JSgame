// 金币对象
var _yinfu1 = new Image();
_yinfu1.src = "img/yinfu1.png";
_yinfu1.value = 7;
_yinfu1.speed = 5;
var _yinfu2 = new Image();
_yinfu2.src = "img/yinfu2.png";
_yinfu2.value = 5;
_yinfu2.speed = 3;
var _yinfu3 = new Image();
_yinfu3.src = "img/yinfu3.png";
_yinfu3.value = 3;
_yinfu3.speed = 2;
var _yinfu4 = new Image();
_yinfu4.src = "img/yinfu4.png";
_yinfu4.value = 1;
_yinfu4.speed = 1;

var heroImg = new Image();
heroImg.src = "img/smiley.png";
 
var bg = new Image();
bg.src = "img/bg.jpg";

var _xDistance = 20;
var _tRest = 30;
var _tTimer;
var _clientW;

// 金币类;
function Money(x,y,speed,img){
    // 每次循环增加的像素数
    this.speed = speed;
    this.x = x;
    this.y = y;
    this.width = img.width;
    this.height = img.height;
    this.img = img;
    this.value = img.value;
}
Money.prototype = {//绘制金币，图像，位置，步幅
    draw:function(ctx){
        ctx.drawImage(this.img,this.x,this.y);
    },
    move:function(){
        this.y += this.speed;
    }
}
// 初始类
function Hero(x,y,img){
    this.grade = 0;//初始得分
    this.life = 10;//最大丢失数 
    this.x = x;
    this.y = y;
    this.img = img;
    this.width = img.width;
    this.height = img.height;
}
Hero.prototype = {
    draw:function(ctx){
        ctx.drawImage(this.img,this.x,this.y);
    },
    touch:function(other){  //碰撞检测 other指丢落对象
    if( this.x + this.width > other.x && this.x < other.x + other.width &&
            this.y + this.height > other.y && this.y < other.y + other.height ){
            this.grade += other.value;
            return true;
    }
        return false;
    }
}
var App = {
    // 对象
    elements:[],//总金币对象
    backImg:bg,//背景定义
    imgs:[_yinfu1,_yinfu2,_yinfu3,_yinfu4],//定义掉落对象
    hero:null,
    // 画布
    canvas:null,
    // 绘制工具
    context:null,
    // 定时器
    timer:null,
    // 速度（更新间隔speed * 10）
    speed:0,
    pause:false,
    // 绘制对象
    draw:function(){
        // 清屏
        this.context.clearRect(0,0,this.canvas.width,canvas.height);
        // 绘制背景
        this.context.drawImage(this.backImg,0,0);
        // 绘制人物
        this.hero.draw(this.context);
        // 绘制金币
        for(var i=0;i<this.elements.length;i++){
            var o = this.elements[i];//当前对象
            // 清理屏幕外的对象
            if(o.x > this.canvas.width || o.x < 0 || o.y > this.canvas.height || o.y < 0){
                    this.elements.splice(i,1);
                    this.hero.life--;

                    this.context.textAlign = "left";//绘制Miss文字
                    this.context.font = '20px Arial bold';
                    this.context.fillStyle = "#f39322";
                    this.context.fillText("Miss",o.x,this.canvas.height-30);

            }else if(this.hero.touch(o)){//清除碰撞对象
                    this.elements.splice(i,1);

                    this.context.textAlign = "left";//绘制加分
                    this.context.font = '30px Arial bold';
                    this.context.fillStyle = "#f39322";
                    this.context.fillText("+ "+o.value,o.x+20,this.canvas.height-140);
            }else{
                    o.draw(this.context);
            }
        }
        // 绘制生命值及得分
        this.context.textAlign = "left";
        this.context.font = 'bold 18px Arial';
        this.context.fillStyle = "#fff";
        var missText = 10 - this.hero.life;
        this.context.fillText(missText,107,141);
        this.context.fillText(this.hero.grade,62,141);
        if(_tRest<10){
            this.context.fillText("0"+_tRest,32,141);
        }else{
            this.context.fillText(_tRest,32,141);
        }
        
    },
    // 循环处理
    loop:function(){
        var me = App;
        if(me.pause){
            return;
        }
        for(var i=0;i<me.elements.length;i++){
            me.elements[i].move();
        }
        var chance = Math.random() * 1000;
        // 1/10的对象添加概率
        if(chance < 100){
            var img = me.imgs[parseInt(chance%me.imgs.length)];//随机音符对象
            var x = Math.random()*(me.canvas.width - me.imgs[parseInt(chance%me.imgs.length)].width);//在canvas里显示对象
            var y = 0;
            var speed = img.speed;
            var money = new Money(x,y,speed,img);
            me.addElement(money);
        }
        me.draw();
        if(me.hero.life == 0){
            me.gameOver();
        }
    },
    // 开始游戏
    gameStart:function(id,speed){
        var me = this;
        me.canvas = document.getElementById(id);
        me.context = me.canvas.getContext("2d");
        me.speed = speed;
        me.hero = new Hero((me.canvas.width - heroImg.width)/2,me.canvas.height - heroImg.height,heroImg);
        if(this.timer != null) me.gameOver();
        me.canvas.onmousemove = function(e){
            e = window.event || e;
            var x = e.clientX - me.canvas.offsetLeft - me.hero.width/2;
             
            if(x > 0 && x < me.canvas.width - me.hero.width){
                //me.hero.x = x;//随鼠标移动事件
            }
        }

        me.timer = setInterval(me.loop,me.speed * 10);//循环绘制Canvas
        this.timeAccout();
    },
    // 暂停游戏
    gamePause:function(){
        this.pause = true;
        this.context.textAlign = "center";
        this.context.fillStyle = "#000";
        this.context.font = 'bold 50px Arial';
        this.context.fillText("Pause!",this.canvas.width/2,this.canvas.height/2);
        this.context.font = 'bold 20px Arial';
        this.context.fillText("Press space key to continue...",this.canvas.width/2,this.canvas.height/2 + 40);

        if(_tTimer){
            clearInterval(_tTimer);
        }
    },
    // 结束游戏
    gameOver:function(){
        clearInterval(this.timer);
        this.elements = [];
        this.pause = false;
        this.timer = null;
        this.context.textAlign = "center";
        this.context.fillStyle = "#000";
        this.context.font = 'bold 40px Arial';
        this.context.fillText("Game Over",this.canvas.width/2,this.canvas.height/2);
    },
    // 添加对象
    addElement:function(o){
        this.elements.push(o);
    },
    moveLeft:function(){
        var me = this;
        if(me.hero.x>=0 && me.hero.x<=(this.canvas.width - me.hero.width)){
            me.hero.x -= _xDistance;
            me.hero.x = (me.hero.x<0)?0:me.hero.x;
        }
    },
    moveRight:function(){
        var me = this;
        if(me.hero.x>=0 && me.hero.x<=(this.canvas.width - me.hero.width)){
            me.hero.x += _xDistance;
            me.hero.x = (me.hero.x > this.canvas.width - me.hero.width)?this.canvas.width - me.hero.width:me.hero.x;
        }
    },
    timeAccout:function(){
        _tTimer = setInterval(function(){
            _tRest--;
            if(_tRest<0){
                clearInterval(_tTimer);
                App.gameOver();
            }
        },1000);
    }
}
 
window.onload = function (){
    var can = $("canvas");
    var ctx = $("canvas").getContext("2d");
    ctx.drawImage(bg,0,0);
  ctx.drawImage(heroImg,(can.width - heroImg.width)/2,can.height - heroImg.height);
    ctx.textAlign = "center";
    ctx.fillStyle = "#000";
    ctx.font = 'bold 20px Arial';
    //ctx.fillText("Press space key to start...",can.width/2,can.height/2);
	document.onkeyup = function(e){
        if(e.keyCode != 32){
            if(e.keyCode == 37){
                App.moveLeft();
            }else if(e.keyCode == 39){
                App.moveRight();
            }
            return;
        }
        if(App.timer == null){
            App.gameStart("canvas",5);//开始游戏并设定step值
            $('startBtn').style.display = 'none';
        }else if(App.pause){
            App.pause = false;
            App.timeAccout();
        }else{
            App.gamePause("canvas",10);
        }
        
    }
   _clientW = parseInt(document.body.clientWidth);//获取全局宽
}
function $(id){
    return document.getElementById(id);
}
function StartFn(){
    $('startBtn').style.display = 'none';
    App.gameStart("canvas",5);
}

function CanvasClickFn(){
    if(App.timer != null){
        var e = window.event
        var x = e.clientX - (_clientW-320)/2;
        var y = e.clientY;
        if(x<160){
            App.moveLeft();
        }else{
            App.moveRight();
        }
        //document.title = x+'---'+y;
    }
}
