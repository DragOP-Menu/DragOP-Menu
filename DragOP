var ctx = com.mojang.minecraftpe.MainActivity.currentMainActivity.get();
var mods = new Array();
var lc = {en_US:1};
var l = new Array();
l.push(new Array());//Codes
l.push(new Array());//English
//Multi Language Support
function addLString(code, lang, value){
	l[0].forEach(function(entry, index, array){
		if(entry+"".toLowerCase() == code+"".toLowerCase()){
			l[lang][index] = value;
			}
	}
}
//addLString(code, lc.en_US, value);
addLString("hacks.gamemode", lc.en_US, "Gamemode");
addLString("gm.survival", lc.en_US, "Survival");
addLString("gm.creative", lc.en_US, "Creative");
function getL(){
	//Todo
	return l[1];
}
function getLString(code){
	l[0].forEach(function(entry, index, array){
		if(entry+"".toLowerCase().indexOf(code+"".toLowerCase())>-1){
			try{
			return getL()[index];
			}catch(e){
				ctoast(e);
			}
			}
	}
}
}
mods.push({
	name:"Gamemode",
	isStateMode:function(){return false;},
	extra:Level.getGameMode(),
	onEnable:function(btn){},
	onDisable:function(btn){},
	onClick:function(btn){},
	onRefresh:function(btn){btn.setText(getLString("hacks.gamemode")+": "+(this.extra==0?getLString("gm.survival"):getLString("gm.creative"));}
});
//Menu open Button
var menuBtn;
var moving = false;
var dx = 0;
var dy = 0;
var mPosX = ctx.getWindowManager().getDefaultDisplay().getWidth() / 16 * 5;
//Main Menu
var menu;
var hackBtn = function(hack){
var btn = new android.widget.Button(ctx);

return btn;
};
function showMenu(){
	ctx.runOnUiThread(new java.lang.Runnable({
		run: function() {
			try {
				
				menu = new android.widget.PopupWindow(null, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				menu.setBackgroundDrawable(new android.graphics.drawable.ColorDrawable(android.graphics.Color.TRANSPARENT));
				menu.showAtLocation(ctx.getWindow().getDecorView(), android.view.Gravity.CENTER | android.view.Gravity.TOP, ctx.getWindowManager().getDefaultDisplay().getWidth()/100*80, ctx.getWindowManager().getDefaultDisplay().getHeight());
			}catch(e){
				ctoast(e);
			}
		}}
	));
}
function showMenuBtn(){
	menuBtn = new android.widget.Button(ctx);
	ctx.runOnUiThread(new java.lang.Runnable({
		run: function() {
			try {
				var layout = new android.widget.LinearLayout(ctx);
				layout.setOrientation(1);
				/*if(ghost) {
					menuBtn.setText('');
					menuBtn.setBackgroundColor(android.graphics.Color.parseColor("#01ffff00"));
				} else {
					menuBtn.setText('[SH]');
					menuBtn.setBackgroundDrawable(new android.graphics.drawable.BitmapDrawable(bg));
				}
				*/
				menuBtn.setText('[SH]');
				//menuBtn.setBackgroundDrawable(new android.graphics.drawable.BitmapDrawable(bg));
				
				menuBtn.setOnClickListener(new android.view.View.OnClickListener({
					onClick: function(viewarg) {
						//mainMenu();
						//windowMenu();
						/*exit();*/
						//GUI.dismiss();
						//GUI = null;
					}
				}));
				menuBtn.setOnTouchListener(new android.view.View.OnTouchListener() {
					onTouch: function(view, motionEvent) {
						try {
							if (!moving) return false;
							switch (motionEvent.getAction()) {
								case android.view.MotionEvent.ACTION_DOWN:
									dx = mPosX - motionEvent.getRawX();
									break;
								case android.view.MotionEvent.ACTION_MOVE:
									mPosX = (motionEvent.getRawX() + dx);
									GUI.update(mPosX, 0, -1, -1);
									break;
								case android.view.MotionEvent.ACTION_UP:
								case android.view.MotionEvent.ACTION_CANCEL:
									moving = false;
									break;
							}
						} catch (e) {
							ctoast("Error: " + e);
						}

						return true;
					}
				});
				menuBtn.setOnLongClickListener(new android.view.View.OnLongClickListener() {
					onLongClick: function(v, t) {
						ctx.getSystemService(android.content.Context.VIBRATOR_SERVICE)
							.vibrate(60);
						/*gad();
						configd1();
						advsetd();*/
						moving = true;
						return true;
					}
				});
				menuBtn.getBackground().setAlpha(200);
				layout.addView(menuBtn);
				GUI = new android.widget.PopupWindow(layout, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				GUI.setBackgroundDrawable(new android.graphics.drawable.ColorDrawable(android.graphics.Color.TRANSPARENT));
				GUI.showAtLocation(ctx.getWindow()
					.getDecorView(), android.view.Gravity.LEFT | android.view.Gravity.TOP, ctx.getWindowManager()
					.getDefaultDisplay()
					.getWidth() / 16 * 5, 0);
			} catch (err) {
				ctoast( err);
			}
		}
	}));
}
showMenuBtn();
function ctoast(text, showprefix) {
	try {
		var ctx = com.mojang.minecraftpe.MainActivity.currentMainActivity.get();
		ctx.runOnUiThread(new java.lang.Runnable({
			run: function() {
				var thetoast = android.widget.Toast.makeText(com.mojang.minecraftpe.MainActivity.currentMainActivity.get(), text, android.widget.Toast.LENGTH_LONG);
				var layout = new android.widget.LinearLayout(ctx);
				var msg = new android.widget.TextView(ctx);
				if(prefix || prefix == null)text = "Servicehack: "+text;
				msg.setText(text);
				msg.setGravity(android.view.Gravity.CENTER);
				msg.setTextSize(20);
                msg.setPadding(10,10,10,10);
                msg.setTextColor(android.graphics.Color.WHITE);
				layout.addView(msg);
				//layout.setBackground(new android.graphics.drawable.BitmapDrawable(bg));
				thetoast.setView(layout);
				thetoast.show();
			}
		}));
	} catch (e) {
		print(e);
	}
}
