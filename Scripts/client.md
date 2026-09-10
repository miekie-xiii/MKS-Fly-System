```
# MKS AF v1.0.0
# Client Script 
# Miekie KrunkerScript Architecture Framework

# -MKS Fly System-
bool jPr=false;
num jTm=0;
num jCnt=0;
num jCd=0;

bool tog=false;
bool isFlying=false;

bool wPr=false;
bool s=false;
num sTm=0;
num sDir=0;
bool sprint=false;

# Player update
public action onPlayerUpdate(str id,num delta,obj inputs) {
 GAME.log(toStr(tog)+" | "+toStr(isFlying));

 obj p=GAME.PLAYERS.findByID(id);
 if(!notEmpty p||!(bool)p.isYou) {
  return;
 }

 num yaw=(num)p.rotation.x;
 num pitch=(num)p.rotation.y;
 num movDir=(num)inputs.movDir;

 num x=0;
 num y=0.0015;
 num z=0;

 bool jump=(bool)inputs.jump;
 bool ground=(bool)p.onGround;

 bool d=(str)inputs.movDir!="undefined"&&(
  movDir==0||
  movDir==-Math.PI/2||
  movDir==Math.PI/2||
  movDir==Math.PI||
  movDir==-Math.PI/4||
  movDir==-3*Math.PI/4||
  movDir==Math.PI/4||
  movDir==3*Math.PI/4
 );

 num now=GAME.TIME.now();

 # Reset flight state when flight is unavailable
 if(!isFlying) {
  jPr=false;
  jTm=0;
  jCnt=0;
  jCd=0;
  tog=false;
  wPr=false;
  s=false;
  sTm=0;
  sDir=0;
  sprint=false;
  return;
 }

 # Reset jump state when grounded
 if(ground) {
  jPr=false;
  jTm=0;
  jCnt=0;
 }

 # Double-tap jump to toggle flying
 if(jump&&!jPr&&!ground) {
  jPr=true;

  if(jCnt==0) {
   jCnt=1;
   jTm=now;
  }
  else if(now-(num)jTm<400) {
   jCnt=0;
   jTm=0;
   tog=(bool)!tog;

   GAME.NETWORK.send("fJ",{j:tog});

   wPr=false;
   s=false;
   sTm=0;
   sDir=0;
   sprint=false;
  }
  else {
   jCnt=1;
   jTm=now;
  }
 }

 if(!jump) {
  jPr=false;
 }

 # Double-tap movement to sprint
 if(!d) {
  wPr=false;

  if((bool)sprint) {
   sprint=false;
  }
 }
 else if(!wPr) {
  if(s&&movDir==sDir&&now-sTm<400) {
   sprint=true;
  }
  else {
   s=true;
   sTm=now;
   sDir=movDir;
  }

  wPr=true;
 }

 if(s&&now-sTm>=400) {
  s=false;
 }

 num speed=sprint?0.30:0.10;

 if(!tog) {
  return;
 }

 # Calculate movement
 if((str)inputs.movDir!="undefined") {
  num a=yaw+Math.PI-movDir-Math.PI/2;
  num c=speed;

  x=Math.sin(a)*c;
  z=Math.cos(a)*c;
  y=Math.sin(pitch)*(0-Math.sin(movDir))*speed;
 }

 if(!ground&&jump) {
  y=0.12;
 }

 if((bool)inputs.crouch) {
  y=-0.12;
 }

 # Apply velocity smoothing
 num accel=0.01;
 num decel=0.01;

 if((num)p.velocity.x<x) {
  (num)p.velocity.x+=accel;

  if((num)p.velocity.x>x) {
   p.velocity.x=x;
  }
 }
 else if((num)p.velocity.x>x) {
  (num)p.velocity.x-=decel;

  if((num)p.velocity.x<x) {
   p.velocity.x=x;
  }
 }

 if((num)p.velocity.y<y) {
  (num)p.velocity.y+=accel;

  if((num)p.velocity.y>y) {
   p.velocity.y=y;
  }
 }
 else if((num)p.velocity.y>y) {
  (num)p.velocity.y-=decel;

  if((num)p.velocity.y<y) {
   p.velocity.y=y;
  }
 }

 if((num)p.velocity.z<z) {
  (num)p.velocity.z+=accel;

  if((num)p.velocity.z>z) {
   p.velocity.z=z;
  }
 }
 else if((num)p.velocity.z>z) {
  (num)p.velocity.z-=decel;

  if((num)p.velocity.z<z) {
   p.velocity.z=z;
  }
 }
}

# Render
public action render(num delta) {
 if(isFlying) {
  obj size=GAME.OVERLAY.getSize();

  num x=(num)size.width/2;
  num y=(num)size.height-20;

  GAME.OVERLAY.drawText(
   "Double tap SPACE to fly. Double tap W A S D to boost",
   x,
   y,
   0,
   12,
   "center",
   "#FFFFFF",
   0.8
  );
 }
}

# Client receives network message
public action onNetworkMessage(str id,obj data) {
 if(id=="fl"||id=="tg") {
  if(id=="fl") {
   isFlying=(bool)data.f;
  }

  tog=(bool)data.t;

  jPr=false;
  jTm=0;
  jCnt=0;
  jCd=0;

  wPr=false;
  s=false;
  sTm=0;
  sDir=0;
  sprint=false;

  return;
 }

 if(id=="sp") {
  sprint=(bool)data.s;
  return;
 }
}
# -MKS Fly System-
```