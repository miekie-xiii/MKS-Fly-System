```
# MKS AF v1.0.0
# Server Script 
# Miekie KrunkerScript Architecture Framework

# -MKS Fly System-
obj[] flyIDs=obj[];

action netSd(str id,obj d,str pId) {
 GAME.NETWORK.send(id,d,pId);
}

action chFly(str id) {
 bool found=false;

 for(num i=0;i<lengthOf flyIDs;i++) {
  if((str)flyIDs[i].id==id) {
   obj f=flyIDs[i];

   f.tog=false;
   f.jPr=false;
   f.jTm=0;
   f.jCnt=0;
   f.jCd=0;
   f.wPr=false;
   f.s=false;
   f.sTm=0;
   f.sDir=0;
   f.sprint=false;

   found=true;
   break;
  }
 }

 if(!found) {
  addTo flyIDs {
   id:id,
   tog:false,
   jPr:false,
   jTm:0,
   jCnt:0,
   jCd:0,
   wPr:false,
   s:false,
   sTm:0,
   sDir:0,
   sprint:false
  };
 }

 netSd("fl",{f:true,t:false},id);
 netSd("sp",{s:false},id);
}

# Player spawns in
public action onPlayerSpawn(str id) {
 chFly(id);
}

# Player update
public action onPlayerUpdate(str id,num delta,obj inputs) {
 obj p=GAME.PLAYERS.findByID(id);
 if(!notEmpty p) {
  return;
 }

 num yaw=(num)p.rotation.x;
 num pitch=(num)p.rotation.y;
 num movDir=(num)inputs.movDir;

 num x=0;
 num y=0.0015;
 num z=0;
 num i=0;

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

 while(i<lengthOf flyIDs) {
  if((str)flyIDs[i].id==id) {
   break;
  }

  i++;
 }

 if(i==lengthOf flyIDs) {
  return;
 }

 obj f=flyIDs[i];

 # Reset jump state when grounded
 if(ground) {
  f.jPr=false;
  f.jCnt=0;
  f.jTm=0;
 }

 # Double-tap jump to toggle flying
 if(jump&&!f.jPr&&!ground) {
  f.jPr=true;

  if((num)f.jCnt==0) {
   f.jCnt=1;
   f.jTm=now;
  }
  else if(now-(num)f.jTm<400) {
   f.jCnt=0;
   f.jTm=0;
   f.tog=(bool)!f.tog;

   netSd("tg",{t:f.tog},id);
   netSd("sp",{s:false},id);

   f.sprint=false;
   f.wPr=false;
   f.s=false;
   f.sTm=0;
   f.sDir=0;
  }
  else {
   f.jCnt=1;
   f.jTm=now;
  }
 }

 if(!jump) {
  f.jPr=false;
 }

 # Double-tap movement to sprint
 if(!d) {
  f.wPr=false;

  if((bool)f.sprint) {
   f.sprint=false;
   netSd("sp",{s:false},id);
  }
 }
 else if(!f.wPr) {
  if((bool)f.s&&movDir==(num)f.sDir&&now-(num)f.sTm<400) {
   f.sprint=true;
   netSd("sp",{s:true},id);
  }
  else {
   f.s=true;
   f.sTm=now;
   f.sDir=movDir;
  }

  f.wPr=true;
 }

 if((bool)f.s&&now-(num)f.sTm>=400) {
  f.s=false;
 }

 num speed=(bool)f.sprint?0.30:0.10;

 GAME.log(toStr(f.tog)+" | "+toStr(flyIDs[i]));

 if(!f.tog) {
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

# Server receives network message
public action onNetworkMessage(str id,obj data,str pID) {
 if(id=="fJ") {
  bool fnd=false;

  for(num i=0;i<lengthOf flyIDs;i++) {
   if((str)flyIDs[i].id==pID) {
    obj f=flyIDs[i];
    fnd=true;

    if((str)data.j!="undefined") {
     f.tog=(bool)data.j;
     f.jTm=0;
     f.jCnt=0;
     f.wPr=false;
     f.s=false;
     f.sTm=0;
     f.sDir=0;
     f.sprint=false;

     netSd("tg",{t:f.tog},pID);
     netSd("sp",{s:false},pID);
    }
    else if((str)data.s!="undefined") {
     f.sprint=(bool)data.s;
     netSd("sp",{s:f.sprint},pID);
    }

    return;
   }
  }
 }
}
# -MKS Fly System-
```