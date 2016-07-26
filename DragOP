
var ctx = com.mojang.minecraftpe.MainActivity.currentMainActivity.get();
var LinearLayout = android.widget.LinearLayout;
var GUI;
var dialog;
var searchQ = ""; //Current search query
var tick1 = 0;
var search;
var Launcher = {
	isBlockLauncher: function () {
		return(ctx.getPackageName() == "net.zhuoweizhang.mcpelauncher" || ctx.getPackageName() == "net.zhuoweizhang.mcpelauncher.pro");
	},
	isToolbox: function () {
		return ctx.getPackageName() == "io.mrarm.mctoolbox";
	}
};

var DragOP = {
	mods: new Array(),
	commands: new Array(),
	isDev: true,
	inGame: false
};
var CommandManager = {
	cmdModules: new Array(),
	cmdNames: new Array(),
	onCommand: function (cmd) {
		cmd = cmd.split(" ");
		var command = cmd.shift().toLowerCase();
		var args = cmd;
		var module = null;
		var found = false;
		this.cmdNames.forEach(function (entry, index) {
			if(found == false) {
				entry.forEach(function (entry2) {
					if(entry2.toLowerCase() == command) {
						found = true;
						module = CommandManager.cmdModules[index];

					}
				});
			}
		});
		if(module != null)
			module.onCall(args);
		else
			DragOP.cmsg("Command \"" + command + "\" not found");
	},
	registerCommand: function (module) {
		this.cmdModules.push(module);
		this.cmdNames.push(module.alias);
	}
};

var FriendManager = {
	all: new org.json.JSONArray(),
	isFriend: function (name) {
		var is = false;
		if(name == null) return false;
		var cname = Utils.Text.clean(name.toString().toLowerCase());

		for(var i = 0; i < this.all.length(); i++) {
			if(cname.toString().toLowerCase() == this.all.getString(i).toLowerCase()) is = true;
		}
		return is;
	},
	loadFromFile: function () {
		try {
			var file = new java.io.File(android.os.Environment.getExternalStorageDirectory() + "/DragOP/", "friends.dat");
			var readed = (new java.io.BufferedReader(new java.io.FileReader(file)));
			var data = new java.lang.StringBuilder();
			var string;
			while((string = readed.readLine()) != null) {
				data.append(string);

			}
			try {
				this.all = new org.json.JSONArray(data.toString());
			} catch(e) {
				DragOP.ctoast("Friend data corrupt. Deleting your friend list(" + e);
			}
		} catch(e) {
			//Seems like theres no file
		}
	},
	saveToFile: function () {
		var dir = new java.io.File(android.os.Environment.getExternalStorageDirectory() + "/DragOP");
		if(!dir.exists()) dir.mkdir();
		var file = new java.io.File(android.os.Environment.getExternalStorageDirectory() + "/DragOP/", "friends.dat");
		if(!file.exists()) file.createNewFile();
		var stream = new java.io.FileOutputStream(file);
		try {
			stream.write(this.all.toString().getBytes());
		} finally {
			stream.close();
		}
	},
	addFriend: function (name) {
		this.all.put(name);
		this.saveToFile();
	},
	removeFriend: function (name) {
		var tempall = new org.json.JSONArray();
		for(var i = 0; i < this.all.length(); i++) {
			if(this.all.getString(i).toLowerCase() != name.toString().toLowerCase()) tempall.put(this.all.getString(i));
		}
		this.all = tempall;
		this.saveToFile();
	}
};
FriendManager.loadFromFile();
var lc = {
	en_US: 1,
	de_DE: 2
};
var ModuleType = {
	mod: 1,
	special: 2,
	command: 3,
	cmd: 3,
	toName: function (type) {
		switch(type) {
		case ModuleType.mod:
			return DragOP.getLString("moduletype.mod");
			break;
		case ModuleType.special:
			return DragOP.getLString("moduletype.special");
			break;
		case ModuleType.command:
			return DragOP.getLString("moduletype.command");
			break;
		default:
			return "unknown";
		}
	}
};
var l = new Array();
l.push(new Array()); //Codes
l.push(new Array()); //English
l.push(new Array()); //German
//Multi Language Support
var BypassMode = {
	DEFAULT: 0,
	LBSG: 1
};
var SpeedMode = {
	DEFAULT: 0,
	BHOP: 1,
	LONGJUMP: 2
};
var getStyledBtnBackground = function (state, toggleable) {
	var bg = android.graphics.drawable.GradientDrawable();
	bg.setCornerRadius(1);
	bg.setColor(android.graphics.Color.argb(80, 0, 0, 0));
	if(state) bg.setColor(android.graphics.Color.argb(210, 0, 200, 0));
	if(toggleable == false) bg.setColor(android.graphics.Color.argb(210, 230, 150, 30));
	bg.setShape(android.graphics.drawable.GradientDrawable.RECTANGLE);
	bg.setStroke(dip2px(2), android.graphics.Color.argb(230, 0, 0, 0));
};
var Themes = {
	DEFAULT: {
		font: android.os.Build.VERSION.SDK_INT >= 17 ? android.graphics.Typeface.create("sans-serif-light", android.graphics.Typeface.NORMAL) : android.graphics.Typeface.DEFAULT,
		ModButton: {

			Neutral: {
				textColor: android.graphics.Color.WHITE,
				background: getStyledBtnBackground(false, false)
			},
			Activated: {
				textColor: android.graphics.Color.WHITE,
				background: getStyledBtnBackground(true, true)
			},
			Deactivated: {
				textColor: android.graphics.Color.WHITE,
				background: getStyledBtnBackground(false, true)
			}
		}

	}
};
var Utils = {
	bypassMode: BypassMode.DEFAULT,
	speedMode: SpeedMode.DEFAULT,
	online: false,
	flyTick: 0,
	modsCount: 0,
	currentSearchCount: 0,
	font: android.os.Build.VERSION.SDK_INT >= 17 ? android.graphics.Typeface.create("sans-serif-light", android.graphics.Typeface.NORMAL) : android.graphics.Typeface.DEFAULT,
	Render: {

		getFloatBuffer: function (fArray) {
			var bBuffer = java.nio.ByteBuffer.allocateDirect(fArray.length * 4);
			bBuffer.order(java.nio.ByteOrder.nativeOrder());

			var fBuffer = bBuffer.asFloatBuffer();
			fBuffer.put(fArray);
			fBuffer.position(0);
			return fBuffer;
		},
		getShortBuffer: function (sArray) {
			var bBuffer = java.nio.ByteBuffer.allocateDirect(sArray.length * 2);
			bBuffer.order(java.nio.ByteOrder.nativeOrder());

			var sBuffer = bBuffer.asShortBuffer();
			sBuffer.put(sArray);
			sBuffer.position(0);
			return sBuffer;
		},
		renderer: null,
		glSurface: null,
		fov: 90,
		init: function () {
			var options = Utils.File.getTextFromFile(new java.io.File(android.os.Environment.getExternalStorageDirectory() + "/games/com.mojang/minecraftpe/", "options.txt"));

			options = options.split("\n");
			options.forEach(function (entry) {
				var suboption = entry.split(":");
				if(suboption[0] == "gfx_field_of_view") {
					Utils.Render.fov = suboption[1];

				}
			});
			this.renderer = new android.opengl.GLSurfaceView.Renderer({
				onSurfaceCreated: function (gl, config) {
					var GL10 = javax.microedition.khronos.opengles.GL10;
					gl.glClearColor(0, 0, 0, 0);
					gl.glShadeModel(GL10.GL_SMOOTH);
					gl.glClearDepthf(1.0);
					gl.glEnable(GL10.GL_DEPTH_TEST);
					gl.glDepthFunc(GL10.GL_LEQUAL);
					gl.glHint(GL10.GL_PERSPECTIVE_CORRECTION_HINT, GL10.GL_NICEST);
					//DragOP.ctoast("created");
				},
				onSurfaceChanged: function (gl, width, height) {
					//DragOP.ctoast("Changing...");
					var GL10 = javax.microedition.khronos.opengles.GL10;
					gl.glMatrixMode(GL10.GL_PROJECTION);
					gl.glLoadIdentity();
					android.opengl.GLU.gluPerspective(gl, Utils.Render.fov, width / height, 0.1, 100);
					gl.glMatrixMode(GL10.GL_MODELVIEW);
					gl.glLoadIdentity();
					//DragOP.ctoast("changed");
				},
				onDrawFrame: function (gl) {
					//DragOP.ctoast("Drawing...");
					var GL10 = javax.microedition.khronos.opengles.GL10;
					gl.glClear(GL10.GL_COLOR_BUFFER_BIT | GL10.GL_DEPTH_BUFFER_BIT);
					gl.glLoadIdentity();
					gl.glDisable(GL10.GL_LIGHTING);
					if(playerEsp.state == true) {
						var yaw = getYaw() % 360;
						var pitch = getPitch() % 360;
						var eyeX = getPlayerX();
						var eyeY = getPlayerY() + 1;
						var eyeZ = getPlayerZ();

						var dCenterX = Math.sin(yaw / 180 * Math.PI);
						var dCenterZ = Math.cos(yaw / 180 * Math.PI);
						var dCenterY = Math.sqrt(dCenterX * dCenterX + dCenterZ * dCenterZ) * Math.tan((pitch - 180) / 180 * Math.PI);

						var centerX = eyeX - dCenterX;
						var centerZ = eyeZ + dCenterZ;
						var centerY = eyeY - dCenterY;

						android.opengl.GLU.gluLookAt(gl, eyeX, eyeY, eyeZ, centerX, centerY, centerZ, 0, 1.0, 0);

						DragOP.mods.forEach(function (entry, index, array) {
							try {
								entry.onRender(gl);
								//gl.glTranslatef(0,0,0);
							} catch(e) {}
						});
					}
				}
			});
			ctx.runOnUiThread(new java.lang.Runnable( {
				run: function () {
					Utils.Render.glSurface = new android.opengl.GLSurfaceView(ctx);
					Utils.Render.glSurface.setZOrderOnTop(true);
					Utils.Render.glSurface.setEGLConfigChooser(8, 8, 8, 8, 16, 0);
					Utils.Render.glSurface.getHolder().setFormat(android.graphics.PixelFormat.TRANSLUCENT);
					Utils.Render.glSurface.setRenderer(Utils.Render.renderer);
					new java.lang.Thread(new java.lang.Runnable( {
						run: function() {
							java.lang.Thread.sleep(1000);
							ctx.runOnUiThread(new java.lang.Runnable( {
								run: function () {
									ctx.getWindow().getDecorView().addView(Utils.Render.glSurface);
								}
							}));
						}
					})).start();
				}
			}));
			
		},
		drawBox: function (gl, x, y, z, x2, y2, z2) {
			var GL10 = javax.microedition.khronos.opengles.GL10;
			var size = new Array(x2, y2, z2);
			var vertices = [
				0, 0, 0,
				size[0], 0, 0,
				0, 0, size[2],
				size[0], 0, size[2],

				0, size[1], 0,
				size[0], size[1], 0,
				0, size[1], size[2],
				size[0], size[1], size[2]
			];
			var vertexBuffer = Utils.Render.getFloatBuffer(vertices);
			var indices = [
				0, 1,
				0, 2,
				0, 4,

				3, 1,
				3, 2,
				3, 7,

				5, 4,
				5, 7,
				5, 1,

				6, 4,
				6, 7,
				6, 2
			];
			var indexBuffer = Utils.Render.getShortBuffer(indices);
			gl.glTranslatef(x, y, z);
			gl.glFrontFace(GL10.GL_CCW);
			//gl.glEnable(GL10.GL_CULL_FACE);
			//gl.glCullFace(GL10.GL_BACK);
			gl.glEnable(GL10.GL_BLEND);
			gl.glBlendFunc(GL10.GL_SRC_ALPHA, GL10.GL_ONE_MINUS_SRC_ALPHA);
			gl.glLineWidth(4);
			gl.glColor4f(0.0, 1.0, 0.0, 0.4);
			gl.glEnableClientState(GL10.GL_VERTEX_ARRAY);
			gl.glVertexPointer(3, GL10.GL_FLOAT, 0, vertexBuffer);
			gl.glDrawElements(GL10.GL_LINES, indices.length, GL10.GL_UNSIGNED_SHORT, indexBuffer);
			gl.glTranslatef(-x, -y, -z);
		}
	},
	Theme: {
		current: Themes.DEFAULT
	},
	Screen: {
		currentScreen: "",
		INGAME: "hud_screen"
	},
	File: {
		dragopDir: android.os.Environment.getExternalStorageDirectory() + "/DragOP/",
		getTextFromFile: function (file) {

			var readed = (new java.io.BufferedReader(new java.io.FileReader(file)));
			var data = new java.lang.StringBuilder();
			var string;
			while((string = readed.readLine()) != null)
				data.append(string + "\n");
			return data.toString();
		},
		saveTextToFile: function (file, text) {
			if(!file.exists()) file.createNewFile();
			var bytes = java.lang.reflect.Array.newInstance(java.lang.Byte.TYPE, text.length());
			for(var i = 0; i < text.length(); i++) bytes[i] = text.charCodeAt(i);
			var stream = new java.io.FileOutputStream(file);
			try {
				stream.write(bytes);
			} finally  {
				stream.close();
			}
		}
	},
	Base64: {
		encode: function (text) {

			var bytes = java.lang.reflect.Array.newInstance(java.lang.Byte.TYPE, text.length());
			for(var i = 0; i < text.length(); i++) bytes[i] = new java.lang.Byte(new String(text).charCodeAt(i));
			return android.util.Base64.encodeToString(bytes, 0);
		},
		decode: function (text) {
			return android.util.Base64.decode(text, 0);
		}
	},
	Text: {
		clean: function (text) {
			var allColor = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "a", "b", "c", "d", "e", "f", "k", "l", "m", "n", "o", "r"];

			if(text != null) {

				allColor.forEach(function (entry) {
					text = text.replace(new RegExp("\u00A7" + entry, 'g'), "");
				});
				return text;
			} else
				return "";
		}
	},
	MobRender: {
		makeInvi: function (renderer) {
			var model = renderer.getModel();
			model.getPart("body").clear();
			model.getPart("rightArm").clear();
			model.getPart("head").clear();
			model.getPart("leftArm").clear();
			model.getPart("rightLeg").clear();
			model.getPart("leftLeg").clear();
			renderer.name = "godsoft.de.Invi";
		}
	},
	ModSettings: {
		getSlider: function () {
			return new android.widget.SeekBar(ctx);
		}
	},
	Block: {
		isLiquid: function (id) {
			if(id >= 8 && id <= 11) return true;
			return false;
		}
	},
	Player: {
		isInWater: function () {
			if(Utils.Block.isLiquid(getTile(getPlayerX(), getPlayerY() - 1.65, getPlayerZ()))) return true;
			return false;
		},
		onGround: function () {
			var y = getPlayerY();
			while(y > 1) y -= 1;

			if((Math.round(y * 100) >= 61 && Math.round(y * 100) <= 63) && getTile(getPlayerX(), getPlayerY() - 1.65, getPlayerZ()) != 0 && !Utils.Block.isLiquid(getTile(getPlayerX(), getPlayerY() - 1.65, getPlayerZ()))) return true;
			if((Math.round(y * 100) >= 11 && Math.round(y * 100) <= 13) && getTile(getPlayerX(), getPlayerY() - 1.65, getPlayerZ()) != 0 && !Utils.Block.isLiquid(getTile(getPlayerX(), getPlayerY() - 1.65, getPlayerZ()))) return true;
			return false;
		},
		isCollidedHorizontally: function () {
			var x = getPlayerX();
			var z = getPlayerZ();
			var blockX = Math.round(x - 0.5);
			var blockZ = Math.round(z - 0.5);
			while(x < 1) x += 1;
			while(z < 1) z += 1;
			while(x > 1) x -= 1;
			while(z > 1) z -= 1;

			if(Math.round(x * 100) == 31) x -= 0.01;
			if(Math.round(z * 100) == 31) z -= 0.01;
			if(Math.round(x * 100) == 69) x += 0.01;
			if(Math.round(z * 100) == 69) z += 0.01;
			if(Math.round(x * 100) == 30) blockX--;
			if(Math.round(z * 100) == 30) blockZ--;
			if(Math.round(x * 100) == 70) blockX++;
			if(Math.round(z * 100) == 70) blockZ++;
			//clientMessage(blockX+";"+blockZ);
			if(getTile(blockX, getPlayerY(), blockZ) == 0 && getTile(blockX, getPlayerY() - 1, blockZ) == 0) return false;

			if(Block.getDestroyTime(getTile(blockX, getPlayerY() - 1, blockZ)) <= 0.1 && Block.getDestroyTime(getTile(blockX, getPlayerY(), blockZ)) <= 0.1) return false;

			if(Math.round(x * 100) == 30 || Math.round(x * 100) == 70) return true;
			if(Math.round(z * 100) == 30 || Math.round(z * 100) == 70) return true;
			return false;
		}
	},
	Vel: {
		lastX: 0,
		lastY: 0,
		lastZ: 0,
		calculateSpeed: function () {
			return Math.sqrt(Math.pow(Entity.getVelX(getPlayerEnt()), 2) + Math.pow(Entity.getVelZ(getPlayerEnt()), 2));
		}
	},
	Pos: {
		lastX: 0,
		lastY: 0,
		lastZ: 0
	},
	Entity: {
		getAll: function () {
			if(Launcher.isToolbox()) {
				return Entity.getAll();
			} else {
				return Utils.Entity.allEntitys;
			}
		},
		targettedMobs: [true, true],
		/*first mobs second players*/
		allEntitys: new Array(),
		charEnts: new Array(),
		crosshairAimAt: function (ent, pos) {
			if(ent != null) {
				var x = Entity.getX(ent) - getPlayerX();
				var y = Entity.getY(ent) - getPlayerY();
				var z = Entity.getZ(ent) - getPlayerZ();
				if(pos != null && pos instanceof Array) {

					x = Entity.getX(ent) - pos[0];
					y = Entity.getY(ent) - pos[1];
					z = Entity.getZ(ent) - pos[2];
				}
				if(Entity.getEntityTypeId(ent) != 63)
					y += 0.5;
				var a = 0.5 + Entity.getX(ent);
				var b = Entity.getY(ent);
				var c = 0.5 + Entity.getZ(ent);
				var len = Math.sqrt(x * x + y * y + z * z);
				var y = y / len;
				var pitch = Math.asin(y);
				pitch = pitch * 180.0 / Math.PI;
				pitch = -pitch;
				var yaw = -Math.atan2(a - (Player.getX() + 0.5), c - (Player.getZ() + 0.5)) * (180 / Math.PI);
				if(pitch < 89 && pitch > -89) {
					Entity.setRot(Player.getEntity(), yaw, pitch);
				}
			}
		},
		bowAimAt: function (ent) {
			var velocity = 1;
			var posX = Entity.getX(ent) - Player.getX();
			var posY = Entity.getEntityTypeId(ent) == EntityType.PLAYER ? Entity.getY(ent) - Player.getY() : Entity.getY(ent) + 1 - Player.getY();
			var posZ = Entity.getZ(ent) - Player.getZ();
			var yaw = (Math.atan2(posZ, posX) * 180 / Math.PI) - 90;
			var y2 = Math.sqrt(posX * posX + posZ * posZ);
			var g = 0.007;
			var tmp = (velocity * velocity * velocity * velocity - g * (g * (y2 * y2) + 2 * posY * (velocity * velocity)));
			var pitch = -(180 / Math.PI) * (Math.atan((velocity * velocity - Math.sqrt(tmp)) / (g * y2)));
			if(pitch < 89 && pitch > -89) {
				setRot(Player.getEntity(), yaw, pitch);
			}

		},
		getNearestEntity(maxrange) {
			var mobs = Utils.Entity.getAll();
			//clientMessage(mobs.length);
			var players = Server.getAllPlayers();

			var small = maxrange;
			var ent = null;
			/*if(mobs.length > 500)clientMessage("lag found: "+mobs.length);*/
			var much = 0;

			mobs.forEach(function (entry) {

				var x = Entity.getX(entry) - getPlayerX();
				var y = Entity.getY(entry) - getPlayerY();
				var z = Entity.getZ(entry) - getPlayerZ();

				var dist = Math.sqrt(Math.pow(x, 2) + Math.pow(y, 2) + Math.pow(z, 2));

				var allowed = false;
				if(Utils.Entity.targettedMobs[1] == true && Entity.getEntityTypeId(entry) == 63) allowed = true;
				if(Utils.Entity.targettedMobs[0] == true && Entity.getEntityTypeId(entry) < 63) allowed = true;
				//clientMessage("Im here2");

				//clientMessage("Im here3");
				if(dist < small && dist > 0 && allowed == true && Entity.getHealth(entry) > 0) {
					if(Entity.getNameTag(entry) == "" || FriendManager.isFriend(Entity.getNameTag(entry)) == false) {
						small = dist;
						ent = entry;
					}
				}

			});


			for(var i = 0; i < players.length; i++) {
				var x = Entity.getX(players[i]) - getPlayerX();
				var y = Entity.getY(players[i]) - getPlayerY();
				var z = Entity.getZ(players[i]) - getPlayerZ();
				var allowed = false;
				if(Utils.Entity.targettedMobs[1] == true && Entity.getEntityTypeId(players[i]) == 63) allowed = true;
				if(Utils.Entity.targettedMobs[0] == true && Entity.getEntityTypeId(players[i]) < 63) allowed = true;
				var dist = Math.sqrt(Math.pow(x, 2) + Math.pow(y, 2) + Math.pow(z, 2));
				if(FriendManager.isFriend(Entity.getNameTag(players[i])) == true) allowed = false;
				if(dist < small && dist > 0 && allowed == true && Entity.getHealth(players[i]) >= 1) {
					small = dist;
					ent = players[i];
				}
			}

			return ent;
		}
	}
};
var _0x1c92 = ["\x35\x20\x66\x3D\x7B\x31\x64\x3A\x30\x2C\x6A\x3A\x31\x70\x2C\x46\x3A\x39\x20\x44\x2E\x58\x2E\x6F\x28\x61\x2E\x6F\x2E\x31\x38\x2C\x22\x52\x2E\x55\x22\x29\x2C\x56\x3A\x6B\x28\x29\x7B\x31\x6F\x28\x37\x2E\x46\x2E\x31\x7A\x28\x29\x29\x7B\x59\x7B\x35\x20\x31\x3D\x61\x2E\x6F\x2E\x31\x65\x28\x37\x2E\x46\x29\x3B\x31\x3D\x31\x2E\x45\x28\x22\x5C\x6E\x22\x2C\x22\x22\x29\x3B\x31\x3D\x61\x2E\x76\x2E\x4C\x28\x31\x29\x3B\x35\x20\x62\x3D\x22\x22\x2C\x63\x3D\x30\x3B\x64\x28\x35\x20\x69\x20\x4D\x20\x31\x29\x63\x2B\x2B\x3B\x64\x28\x35\x20\x69\x3D\x30\x3B\x69\x3C\x63\x3B\x69\x2B\x2B\x29\x7B\x62\x2B\x3D\x50\x2E\x4B\x28\x31\x5B\x69\x5D\x29\x7D\x31\x3D\x62\x3B\x31\x3D\x31\x2E\x45\x28\x22\x5C\x6E\x22\x2C\x22\x22\x29\x3B\x31\x3D\x61\x2E\x76\x2E\x4C\x28\x31\x29\x3B\x35\x20\x62\x3D\x22\x22\x2C\x63\x3D\x30\x3B\x64\x28\x35\x20\x69\x20\x4D\x20\x31\x29\x63\x2B\x2B\x3B\x64\x28\x35\x20\x69\x3D\x30\x3B\x69\x3C\x63\x3B\x69\x2B\x2B\x29\x7B\x62\x2B\x3D\x50\x2E\x4B\x28\x31\x5B\x69\x5D\x29\x7D\x31\x3D\x62\x3B\x31\x3D\x31\x2E\x45\x28\x22\x5C\x6E\x22\x2C\x22\x22\x29\x3B\x31\x3D\x39\x20\x48\x5B\x22\x47\x22\x5D\x2E\x5A\x28\x31\x29\x3B\x66\x2E\x31\x30\x28\x31\x31\x28\x31\x2E\x31\x33\x28\x22\x31\x34\x22\x29\x29\x29\x3B\x31\x3D\x31\x2E\x31\x68\x28\x22\x31\x36\x22\x29\x3B\x31\x3D\x61\x2E\x76\x2E\x4C\x28\x31\x29\x3B\x35\x20\x62\x3D\x22\x22\x2C\x63\x3D\x30\x3B\x64\x28\x35\x20\x69\x20\x4D\x20\x31\x29\x63\x2B\x2B\x3B\x64\x28\x35\x20\x69\x3D\x30\x3B\x69\x3C\x63\x3B\x69\x2B\x2B\x29\x7B\x62\x2B\x3D\x50\x2E\x4B\x28\x31\x5B\x69\x5D\x29\x7D\x31\x3D\x62\x3B\x31\x3D\x31\x2E\x45\x28\x22\x5C\x6E\x22\x2C\x22\x22\x29\x3B\x31\x3D\x39\x20\x48\x2E\x47\x2E\x31\x61\x28\x31\x29\x3B\x37\x2E\x6A\x3D\x39\x20\x4A\x28\x29\x3B\x64\x28\x35\x20\x69\x3D\x30\x3B\x69\x3C\x31\x2E\x53\x28\x29\x3B\x69\x2B\x2B\x29\x7B\x37\x2E\x6A\x2E\x31\x63\x28\x31\x2E\x31\x33\x28\x69\x29\x29\x7D\x7D\x54\x28\x65\x29\x7B\x31\x67\x2E\x31\x43\x28\x22\x66\x20\x31\x69\x20\x31\x6A\x2E\x5C\x31\x6B\x20\x66\x20\x31\x6D\x22\x29\x3B\x37\x2E\x46\x2E\x31\x6E\x28\x29\x3B\x37\x5B\x78\x5B\x22\x38\x22\x5D\x5B\x30\x5D\x2E\x70\x28\x29\x2B\x28\x71\x2E\x72\x28\x67\x29\x3D\x3D\x75\x3F\x28\x34\x2E\x6D\x28\x77\x29\x2A\x6C\x29\x3A\x28\x34\x2E\x79\x28\x29\x2A\x33\x2F\x36\x2A\x30\x2B\x34\x2E\x7A\x28\x41\x2E\x42\x29\x29\x29\x2B\x43\x5B\x22\x38\x22\x5D\x5B\x32\x5D\x2B\x22\x73\x22\x2B\x22\x68\x22\x5D\x3D\x30\x3B\x37\x2E\x6A\x3D\x39\x20\x4A\x28\x29\x7D\x7D\x31\x66\x7B\x37\x5B\x78\x5B\x22\x38\x22\x5D\x5B\x30\x5D\x2E\x70\x28\x29\x2B\x28\x71\x2E\x72\x28\x67\x29\x3D\x3D\x75\x3F\x28\x34\x2E\x6D\x28\x77\x29\x2A\x6C\x29\x3A\x28\x34\x2E\x79\x28\x29\x2A\x33\x2F\x36\x2A\x30\x2B\x34\x2E\x7A\x28\x41\x2E\x42\x29\x29\x29\x2B\x43\x5B\x22\x38\x22\x5D\x5B\x32\x5D\x2B\x22\x73\x22\x2B\x22\x68\x22\x5D\x3D\x30\x3B\x37\x2E\x6A\x3D\x39\x20\x4A\x28\x29\x7D\x7D\x2C\x31\x32\x3A\x6B\x28\x29\x7B\x35\x20\x31\x3D\x39\x20\x48\x2E\x47\x2E\x31\x61\x28\x29\x3B\x64\x28\x35\x20\x69\x3D\x30\x3B\x69\x3C\x37\x2E\x6A\x2E\x53\x3B\x69\x2B\x2B\x29\x7B\x31\x2E\x4E\x28\x37\x2E\x6A\x5B\x69\x5D\x29\x7D\x31\x3D\x61\x2E\x76\x2E\x4F\x28\x31\x2E\x31\x35\x28\x29\x29\x3B\x35\x20\x49\x3D\x39\x20\x48\x2E\x47\x2E\x5A\x28\x29\x3B\x49\x2E\x4E\x28\x22\x31\x36\x22\x2C\x31\x29\x3B\x49\x2E\x4E\x28\x22\x31\x34\x22\x2C\x31\x31\x28\x66\x5B\x78\x5B\x22\x38\x22\x5D\x5B\x30\x5D\x2E\x70\x28\x29\x2B\x28\x71\x2E\x72\x28\x67\x29\x3D\x3D\x75\x3F\x28\x34\x2E\x6D\x28\x77\x29\x2A\x6C\x29\x3A\x28\x34\x2E\x79\x28\x29\x2A\x33\x2F\x36\x2A\x30\x2B\x34\x2E\x7A\x28\x41\x2E\x42\x29\x29\x29\x2B\x43\x5B\x22\x38\x22\x5D\x5B\x32\x5D\x2B\x22\x73\x22\x2B\x22\x68\x22\x5D\x29\x29\x3B\x31\x3D\x61\x2E\x76\x2E\x4F\x28\x49\x2E\x31\x35\x28\x29\x29\x3B\x31\x3D\x61\x2E\x76\x2E\x4F\x28\x31\x29\x3B\x61\x2E\x6F\x2E\x31\x6C\x28\x39\x20\x44\x2E\x58\x2E\x6F\x28\x61\x2E\x6F\x2E\x31\x38\x2C\x22\x52\x2E\x55\x22\x29\x2C\x31\x29\x7D\x2C\x31\x37\x3A\x6B\x28\x29\x7B\x35\x20\x74\x3D\x39\x20\x44\x2E\x51\x2E\x31\x39\x28\x39\x20\x44\x2E\x51\x2E\x31\x71\x28\x7B\x31\x72\x3A\x6B\x28\x29\x7B\x31\x73\x28\x67\x29\x7B\x59\x7B\x44\x5B\x22\x51\x22\x5D\x5B\x22\x31\x39\x22\x5D\x2E\x31\x74\x28\x31\x75\x2A\x31\x76\x29\x3B\x66\x5B\x78\x5B\x22\x38\x22\x5D\x5B\x30\x5D\x2E\x70\x28\x29\x2B\x28\x71\x2E\x72\x28\x67\x29\x3D\x3D\x75\x3F\x28\x34\x2E\x6D\x28\x77\x29\x2A\x6C\x29\x3A\x28\x34\x2E\x79\x28\x29\x2A\x33\x2F\x36\x2A\x30\x2B\x34\x2E\x7A\x28\x41\x2E\x42\x29\x29\x29\x2B\x43\x5B\x22\x38\x22\x5D\x5B\x32\x5D\x2B\x22\x73\x22\x2B\x22\x68\x22\x5D\x2B\x3D\x30\x3B\x66\x2E\x31\x32\x28\x29\x7D\x54\x28\x65\x29\x7B\x31\x77\x28\x65\x29\x3B\x31\x78\x28\x65\x29\x7D\x7D\x7D\x7D\x29\x29\x3B\x74\x2E\x31\x79\x28\x29\x7D\x2C\x31\x30\x3A\x6B\x28\x57\x29\x7B\x37\x5B\x78\x5B\x22\x38\x22\x5D\x5B\x30\x5D\x2E\x70\x28\x29\x2B\x28\x71\x2E\x72\x28\x67\x29\x3D\x3D\x75\x3F\x28\x34\x2E\x6D\x28\x77\x29\x2A\x6C\x29\x3A\x28\x34\x2E\x79\x28\x29\x2A\x33\x2F\x36\x2A\x30\x2B\x34\x2E\x7A\x28\x41\x2E\x42\x29\x29\x29\x2B\x43\x5B\x22\x38\x22\x5D\x5B\x32\x5D\x2B\x22\x73\x22\x2B\x22\x68\x22\x5D\x3D\x57\x7D\x2C\x31\x41\x3A\x6B\x28\x29\x7B\x31\x42\x20\x37\x5B\x78\x5B\x22\x38\x22\x5D\x5B\x30\x5D\x2E\x70\x28\x29\x2B\x28\x71\x2E\x72\x28\x67\x29\x3D\x3D\x75\x3F\x28\x34\x2E\x6D\x28\x77\x29\x2A\x6C\x29\x3A\x28\x34\x2E\x79\x28\x29\x2A\x33\x2F\x36\x2A\x30\x2B\x34\x2E\x7A\x28\x41\x2E\x42\x29\x29\x29\x2B\x43\x5B\x22\x38\x22\x5D\x5B\x32\x5D\x2B\x22\x73\x22\x2B\x22\x68\x22\x5D\x7D\x2C\x31\x62\x3A\x6B\x28\x29\x7B\x37\x2E\x56\x28\x29\x3B\x37\x2E\x31\x37\x28\x29\x7D\x7D\x3B", "\x7C", "\x73\x70\x6C\x69\x74", "\x7C\x74\x65\x78\x74\x7C\x7C\x7C\x4D\x61\x74\x68\x7C\x76\x61\x72\x7C\x7C\x74\x68\x69\x73\x7C\x64\x65\x73\x63\x7C\x6E\x65\x77\x7C\x55\x74\x69\x6C\x73\x7C\x6E\x65\x77\x53\x74\x72\x7C\x6C\x65\x6E\x67\x7C\x66\x6F\x72\x7C\x7C\x53\x68\x6F\x70\x7C\x74\x72\x75\x65\x7C\x7C\x7C\x62\x75\x79\x65\x64\x7C\x66\x75\x6E\x63\x74\x69\x6F\x6E\x7C\x34\x39\x32\x7C\x73\x69\x6E\x7C\x7C\x46\x69\x6C\x65\x7C\x74\x6F\x4C\x6F\x77\x65\x72\x43\x61\x73\x65\x7C\x42\x6F\x6F\x6C\x65\x61\x6E\x7C\x76\x61\x6C\x75\x65\x4F\x66\x7C\x7C\x7C\x66\x61\x6C\x73\x65\x7C\x42\x61\x73\x65\x36\x34\x7C\x35\x39\x33\x38\x7C\x67\x6D\x68\x61\x63\x6B\x7C\x72\x61\x6E\x64\x6F\x6D\x7C\x72\x6F\x75\x6E\x64\x7C\x35\x39\x33\x32\x38\x39\x34\x7C\x34\x39\x36\x39\x33\x7C\x73\x70\x65\x65\x64\x7C\x6A\x61\x76\x61\x7C\x72\x65\x70\x6C\x61\x63\x65\x7C\x63\x61\x73\x68\x46\x69\x6C\x65\x7C\x6A\x73\x6F\x6E\x7C\x6F\x72\x67\x7C\x6A\x73\x6F\x6E\x4F\x62\x6A\x7C\x41\x72\x72\x61\x79\x7C\x66\x72\x6F\x6D\x43\x68\x61\x72\x43\x6F\x64\x65\x7C\x64\x65\x63\x6F\x64\x65\x7C\x69\x6E\x7C\x70\x75\x74\x7C\x65\x6E\x63\x6F\x64\x65\x7C\x53\x74\x72\x69\x6E\x67\x7C\x6C\x61\x6E\x67\x7C\x62\x61\x73\x65\x36\x34\x7C\x6C\x65\x6E\x67\x74\x68\x7C\x63\x61\x74\x63\x68\x7C\x64\x61\x74\x7C\x6C\x6F\x61\x64\x43\x61\x73\x68\x7C\x6E\x65\x77\x43\x61\x73\x68\x7C\x69\x6F\x7C\x74\x72\x79\x7C\x4A\x53\x4F\x4E\x4F\x62\x6A\x65\x63\x74\x7C\x73\x65\x74\x43\x61\x73\x68\x7C\x70\x61\x72\x73\x65\x49\x6E\x74\x7C\x73\x61\x76\x65\x43\x61\x73\x68\x7C\x67\x65\x74\x7C\x64\x65\x66\x61\x75\x6C\x74\x74\x7C\x74\x6F\x53\x74\x72\x69\x6E\x67\x7C\x62\x61\x73\x65\x7C\x73\x74\x61\x72\x74\x54\x68\x72\x65\x61\x64\x7C\x64\x72\x61\x67\x6F\x70\x44\x69\x72\x7C\x54\x68\x72\x65\x61\x64\x7C\x4A\x53\x4F\x4E\x41\x72\x72\x61\x79\x7C\x69\x6E\x69\x74\x7C\x70\x75\x73\x68\x7C\x63\x35\x39\x33\x32\x38\x39\x34\x61\x73\x68\x7C\x67\x65\x74\x54\x65\x78\x74\x46\x72\x6F\x6D\x46\x69\x6C\x65\x7C\x65\x6C\x73\x65\x7C\x44\x72\x61\x67\x4F\x50\x7C\x67\x65\x74\x53\x74\x72\x69\x6E\x67\x7C\x64\x61\x74\x61\x7C\x63\x6F\x72\x72\x75\x70\x74\x7C\x6E\x44\x65\x6C\x65\x74\x69\x6E\x67\x7C\x73\x61\x76\x65\x54\x65\x78\x74\x54\x6F\x46\x69\x6C\x65\x7C\x44\x61\x74\x61\x7C\x64\x65\x6C\x65\x74\x65\x7C\x69\x66\x7C\x6E\x75\x6C\x6C\x7C\x52\x75\x6E\x6E\x61\x62\x6C\x65\x7C\x72\x75\x6E\x7C\x77\x68\x69\x6C\x65\x7C\x73\x6C\x65\x65\x70\x7C\x36\x30\x7C\x31\x30\x30\x30\x7C\x70\x72\x69\x6E\x74\x7C\x63\x6C\x69\x65\x6E\x74\x4D\x65\x73\x73\x61\x67\x65\x7C\x73\x74\x61\x72\x74\x7C\x65\x78\x69\x73\x74\x73\x7C\x67\x65\x74\x43\x61\x73\x68\x7C\x72\x65\x74\x75\x72\x6E\x7C\x63\x74\x6F\x61\x73\x74", "", "\x66\x72\x6F\x6D\x43\x68\x61\x72\x43\x6F\x64\x65", "\x72\x65\x70\x6C\x61\x63\x65", "\x5C\x77\x2B", "\x5C\x62", "\x67"];
eval(function (_0x47e1x1, _0x47e1x2, _0x47e1x3, _0x47e1x4, _0x47e1x5, _0x47e1x6) {
	_0x47e1x5 = function (_0x47e1x3) {
		return(_0x47e1x3 < _0x47e1x2 ? _0x1c92[4] : _0x47e1x5(parseInt(_0x47e1x3 / _0x47e1x2))) + ((_0x47e1x3 = _0x47e1x3 % _0x47e1x2) > 35 ? String[_0x1c92[5]](_0x47e1x3 + 29) : _0x47e1x3.toString(36))
	};
	if(!_0x1c92[4][_0x1c92[6]](/^/, String)) {
		while(_0x47e1x3--) {
			_0x47e1x6[_0x47e1x5(_0x47e1x3)] = _0x47e1x4[_0x47e1x3] || _0x47e1x5(_0x47e1x3)
		};
		_0x47e1x4 = [function (_0x47e1x5) {
			return _0x47e1x6[_0x47e1x5]
		}];
		_0x47e1x5 = function () {
			return _0x1c92[7]
		};
		_0x47e1x3 = 1
	};
	while(_0x47e1x3--) {
		if(_0x47e1x4[_0x47e1x3]) {
			_0x47e1x1 = _0x47e1x1[_0x1c92[6]](new RegExp(_0x1c92[8] + _0x47e1x5(_0x47e1x3) + _0x1c92[8], _0x1c92[9]), _0x47e1x4[_0x47e1x3])
		}
	};
	return _0x47e1x1
}(_0x1c92[0], 62, 101, _0x1c92[3][_0x1c92[2]](_0x1c92[1]), 0, {}));

DragOP.addCode = function (code) {
	l[0].push(code);
}
DragOP.setLString = function (code, lang, value) {
		var done = false;
		l[0].forEach(function (entry, index, array) {
			if(entry.toLowerCase() == code.toLowerCase()) {
				l[lang][index] = value;
				done = true;
			}
		});
		if(done) return;
		//DragOP.addCode(code);
		//l[0].push(code+"".toLowerCase());
		//DragOP.setLString(code, lang, value);
	}
	//DragOP.addCode(code);
DragOP.addCode("special.shop");
DragOP.addCode("special.panic");
DragOP.addCode("special.target");
DragOP.addCode("special.friend_manager");
DragOP.addCode("special.bypass");
DragOP.addCode("hacks.gamemode");
DragOP.addCode("hacks.speed");
DragOP.addCode("hacks.aimaura");
DragOP.addCode("hacks.jumpspeed");
DragOP.addCode("hacks.autojump");
DragOP.addCode("hacks.tpaura");
DragOP.addCode("hacks.bowaimbot");
DragOP.addCode("hacks.clicktp");
DragOP.addCode("hacks.longjump");
DragOP.addCode("hacks.flight");
DragOP.addCode("hacks.step");
DragOP.addCode("hacks.jesus");
DragOP.addCode("hacks.nodownglide");
DragOP.addCode("hacks.glide");
DragOP.addCode("hacks.chestTracer");
DragOP.addCode("hacks.criticals");
DragOP.addCode("hacks.coords");
DragOP.addCode("hacks.playerEsp");
DragOP.addCode("hacks.nuker");
DragOP.addCode("gm.survival");
DragOP.addCode("gm.creative");
DragOP.addCode("moduletype.mod");
DragOP.addCode("moduletype.special");
DragOP.addCode("moduletype.command");
DragOP.addCode("moddialog.description");
DragOP.addCode("moddialog.type");
DragOP.addCode("state.on");
DragOP.addCode("state.off");

//DragOP.setLString(code, lc.en_US, value);
DragOP.setLString("special.shop", lc.en_US, "Shop");
DragOP.setLString("special.bypass", lc.en_US, "Bypass");
DragOP.setLString("special.target", lc.en_US, "Target");
DragOP.setLString("special.friend_manager", lc.en_US, "Friend-Manager");
DragOP.setLString("special.panic", lc.en_US, "Panic");
DragOP.setLString("hacks.gamemode", lc.en_US, "Gamemode");
DragOP.setLString("hacks.speed", lc.en_US, "Speed");
DragOP.setLString("hacks.jumpspeed", lc.en_US, "JumpSpeed");
DragOP.setLString("hacks.aimaura", lc.en_US, "AimAura"); //No good German translation for aim aura
DragOP.setLString("hacks.autojump", lc.en_US, "AutoJump");
DragOP.setLString("hacks.tpaura", lc.en_US, "TP-Aura");
DragOP.setLString("hacks.bowaimbot", lc.en_US, "BowAimBot");
DragOP.setLString("hacks.clicktp", lc.en_US, "TapTeleport");
DragOP.setLString("hacks.longjump", lc.en_US, "BunnyHop");
DragOP.setLString("hacks.flight", lc.en_US, "Flight");
DragOP.setLString("hacks.step", lc.en_US, "Step");
DragOP.setLString("hacks.jesus", lc.en_US, "Jesus");
DragOP.setLString("hacks.nodownglide", lc.en_US, "NoDownGlide");
DragOP.setLString("hacks.glide", lc.en_US, "Glide");
DragOP.setLString("hacks.chestTracer", lc.en_US, "ChestTracers");
DragOP.setLString("hacks.criticals", lc.en_US, "Criticals");
DragOP.setLString("hacks.coords", lc.en_US, "Coords");
DragOP.setLString("hacks.playerEsp", lc.en_US, "PlayerESP");
DragOP.setLString("hacks.nuker", lc.en_US, "Nuker");
DragOP.setLString("gm.survival", lc.en_US, "Survival");
DragOP.setLString("gm.creative", lc.en_US, "Creative");
DragOP.setLString("moduletype.mod", lc.en_US, "Mod");
DragOP.setLString("moduletype.special", lc.en_US, "Special");
DragOP.setLString("moduletype.command", lc.en_US, "Command");
DragOP.setLString("moddialog.description", lc.en_US, "Description");
DragOP.setLString("moddialog.type", lc.en_US, "Type");
DragOP.setLString("state.on", lc.en_US, "On");
DragOP.setLString("state.off", lc.en_US, "Off");
//German
DragOP.setLString("hacks.gamemode", lc.de_DE, "Spielmodus");
/*DragOP.setLString("hacks.speed", lc.de_DE, "Schnelligkeit");Schnelligkeit is bad*/
DragOP.setLString("hacks.jumpspeed", lc.de_DE, "HasenTempo"); //Why not using bunnyspeed? :)
DragOP.setLString("hacks.autojump", lc.de_DE, "Automatisches\nSpringen");
DragOP.setLString("hacks.jesus", lc.de_DE, "Wasserlauf");
DragOP.setLString("hacks.flight", lc.de_DE, "Fliegen");
DragOP.setLString("gm.survival", lc.de_DE, "Ã?berleben");
DragOP.setLString("gm.creative", lc.de_DE, "Kreativ");
DragOP.setLString("moduletype.mod", lc.de_DE, "Mod");
DragOP.setLString("moduletype.special", lc.de_DE, "Spezielles");
DragOP.setLString("moduletype.command", lc.de_DE, "Befehl");
DragOP.setLString("moddialog.description", lc.de_DE, "Beschreibung");
DragOP.setLString("moddialog.type", lc.de_DE, "Typ");

DragOP.getL = function () {
	//Todo
	return l[1];
}

DragOP.getLString = function (code) {
	var str = code;
	l[0].forEach(function (entry, index, array) {

		if(entry.toLowerCase()
			.indexOf(code.toLowerCase()) > -1) {
			try {
				str = DragOP.getL()[index];
			} catch(e) {
				try {
					str = l[1][index];
				} catch(e) {
					DragOP.ctoast(e);
				}
			}
		}
	});
	return str;
}
DragOP.cmsg = function (text) {
	clientMessage(ChatColor.GREEN + "[" + ChatColor.BLUE + "Drag" + ChatColor.GOLD + "OP" + ChatColor.GREEN + "]" + ChatColor.GRAY + ": " + ChatColor.YELLOW + text);
}
DragOP.ctoast = function (text, showPrefix) {
	try {
		var ctx = com.mojang.minecraftpe.MainActivity.currentMainActivity.get();
		ctx.runOnUiThread(new java.lang.Runnable({
			run: function () {
				var thetoast = android.widget.Toast.makeText(com.mojang.minecraftpe.MainActivity.currentMainActivity.get(), "" + text, android.widget.Toast.LENGTH_LONG);
				var layout = new android.widget.LinearLayout(ctx);
				var msg = new android.widget.TextView(ctx);
				if(showPrefix || showPrefix == null) text = "DragOP: " + text;
				msg.setText(text);
				msg.setGravity(android.view.Gravity.CENTER);
				msg.setTextSize(20);
				msg.setPadding(10, 10, 10, 10);
				msg.setTextColor(android.graphics.Color.WHITE);
				var bg = new android.graphics.drawable.GradientDrawable();
				bg.setColor(android.graphics.Color.argb(150, 50, 50, 50));
				bg.setStroke(dip2px(5), android.graphics.Color.argb(200, 5, 5, 5));
				bg.setCornerRadius(dip2px(20));
				layout.addView(msg);
				layout.setBackground(bg);
				thetoast.setView(layout);
				thetoast.show();
			}
		}));
	} catch(e) {
		print(e);
	}
}
DragOP.loadModsAnim = function (progress) {
	ctx.runOnUiThread(new java.lang.Runnable({
		run: function () {
			new android.os.Handler()
				.postDelayed(new java.lang.Runnable({
					run: function () {
						ModPE.langEdit("menu.copyright", "DragOP: " + progress + " Modules loaded");
						if(progress < Utils.modsCount) DragOP.loadModsAnim(progress + 1);
					}
				}), 100);
		}
	}));
}
DragOP.getStyledBackground = function () {
	var bg = new android.graphics.drawable.GradientDrawable();
	bg.setCornerRadius(1);
	bg.setColor(android.graphics.Color.argb(90, 255, 255, 255));
	bg.setShape(android.graphics.drawable.GradientDrawable.RECTANGLE);
	bg.setStroke(dip2px(2), android.graphics.Color.argb(210, 0, 0, 0));
	return bg;
}
DragOP.getStyledBtnBackground = function (state, toggleable) {
	var bg = android.graphics.drawable.GradientDrawable();
	bg.setCornerRadius(1);
	bg.setColor(android.graphics.Color.argb(80, 0, 0, 0));
	if(state) bg.setColor(android.graphics.Color.argb(210, 0, 200, 0));
	if(toggleable == false) bg.setColor(android.graphics.Color.argb(210, 230, 150, 30));
	bg.setShape(android.graphics.drawable.GradientDrawable.RECTANGLE);
	bg.setStroke(dip2px(2), android.graphics.Color.argb(230, 0, 0, 0));
	return bg;
}

DragOP.showModDialog = function (mod) {
	ctx.runOnUiThread(new java.lang.Runnable( {
		run: function () {
			try {
				var display = new android.util.DisplayMetrics();
				com.mojang.minecraftpe.MainActivity.currentMainActivity.get()
					.getWindowManager()
					.getDefaultDisplay()
					.getMetrics(display);
				var content = new android.widget.RelativeLayout(ctx);
				content.setId(9472729);
				var contentScroll = new android.widget.ScrollView(ctx);
				contentScroll.setId(492628);
				//default content
				var modTitle = new android.widget.TextView(ctx);
				modTitle.setText(android.text.Html.fromHtml("<u>" + mod.name + "</u>"));
				modTitle.setTextSize(dip2px(20));
				modTitle.setGravity(android.view.Gravity.CENTER);
				modTitle.setTextColor(android.graphics.Color.BLACK);
				modTitle.setTypeface(Utils.font);
				modTitle.setId(94771);
				var modTypeText = new android.widget.TextView(ctx);
				modTypeText.setText(DragOP.getLString("moddialog.type") + ": " + ModuleType.toName(mod.type));
				modTypeText.setGravity(android.view.Gravity.CENTER);
				modTypeText.setTextColor(android.graphics.Color.BLACK);
				modTypeText.setTextSize(dip2px(10));
				modTypeText.setTypeface(Utils.font);
				modTypeText.setId(93922);
				var modDescTitle = new android.widget.TextView(ctx);
				modDescTitle.setText(DragOP.getLString("moddialog.description") + ":");
				modDescTitle.setGravity(android.view.Gravity.CENTER);
				modDescTitle.setTextColor(android.graphics.Color.BLACK);
				modDescTitle.setTextSize(dip2px(11));
				modDescTitle.setTypeface(Utils.font);
				modDescTitle.setId(29582);
				var modDescText = new android.widget.TextView(ctx);
				modDescText.setText(mod.desc);
				modDescText.setGravity(android.view.Gravity.CENTER);
				modDescText.setTextSize(dip2px(10));
				modDescText.setTypeface(Utils.font);
				modDescText.setTextColor(android.graphics.Color.BLACK);
				modDescText.setId(29285);
				//settings
				var modSettings = new android.widget.LinearLayout(ctx);
				modSettings.setOrientation(1);
				if(mod.getSettingsLayout) {
					//modSettings = mod.Layout();
					var params = new android.widget.LinearLayout.LayoutParams(mwidth, android.widget.LinearLayout.LayoutParams.WRAP_CONTENT);
					var line = new android.widget.TextView(ctx);
					line.setText("");
					var gradientLine = new android.graphics.drawable.GradientDrawable();
					gradientLine.setShape(android.graphics.drawable.GradientDrawable.LINE);
					gradientLine.setColor(android.graphics.Color.TRANSPARENT);
					//gradientLine.setCornerRadius(dip2px(100));
					gradientLine.setStroke(dip2px(1), android.graphics.Color.argb(50, 0, 0, 0));
					line.setBackground(gradientLine);
					line.setGravity(android.view.Gravity.CENTER);
					modSettings.addView(line, params);
					var settingText = new android.widget.TextView(ctx);
					settingText.setText("Settings");
					settingText.setGravity(android.view.Gravity.CENTER);
					settingText.setTextColor(android.graphics.Color.BLACK);
					settingText.setTextSize(dip2px(11));
					settingText.setTypeface(Utils.font);
					modSettings.addView(settingText, params);
					var extraParams = new android.widget.LinearLayout.LayoutParams(android.widget.LinearLayout.LayoutParams.MATCH_PARENT, android.widget.LinearLayout.LayoutParams.WRAP_CONTENT);
					modSettings.addView(mod.getSettingsLayout(extraParams));
					//Im thinking about a line (html <hr> tag) and the layout underneath
				}
				//footer
				var closeButton = new styledBtn();
				closeButton.setText("Close");
				closeButton.setPadding(0.5, closeButton.getPaddingTop(), 0.5, closeButton.getPaddingBottom());
				closeButton.setId(10472);
				closeButton.setTypeface(Utils.font);
				closeButton.setTextColor(android.graphics.Color.BLACK);
				closeButton.setTypeface(Utils.font);
				//layout alignement....
				var dialogLayout = new android.widget.RelativeLayout(ctx);
				dialogLayout.setBackgroundDrawable(DragOP.getStyledBackground());
				//dialogLayout.setGravity(android.view.Gravity.CENTER);
				//dialogLayout.setOrientation(LinearLayout.VERTICAL);
				var params = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_TOP);
				dialogLayout.addView(modTitle, params);
				params = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_TOP);
				content.addView(modTypeText, params);
				params = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				params.addRule(android.widget.RelativeLayout.BELOW, modTypeText.getId());
				content.addView(modDescTitle, params);
				params = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				params.addRule(android.widget.RelativeLayout.BELOW, modDescTitle.getId());
				content.addView(modDescText, params);
				params = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				params.addRule(android.widget.RelativeLayout.BELOW, modDescText.getId());
				content.addView(modSettings, params);
				contentScroll.addView(content);
				params = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				params.addRule(android.widget.RelativeLayout.BELOW, modTitle.getId());
				params.addRule(android.widget.RelativeLayout.ABOVE, closeButton.getId());
				contentScroll.setFillViewport(true);
				dialogLayout.addView(contentScroll, params);
				params = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_BOTTOM);
				dialogLayout.addView(closeButton, params);
				//Dialog Stuff
				dialog = new android.app.Dialog(ctx);
				dialog.requestWindowFeature(android.view.Window.FEATURE_NO_TITLE);
				dialog.getWindow()
					.setBackgroundDrawable(new android.graphics.drawable.ColorDrawable(android.graphics.Color.TRANSPARENT));
				dialog.setContentView(dialogLayout);
				dialog.setTitle(mod.name);
				dialog.setOnDismissListener(new android.content.DialogInterface.OnDismissListener( {
					onDismiss: function () {
						showMenu();
					}
				}));
				dialog.show();
				var window = dialog.getWindow();
				window.setLayout(mwidth, display.heightPixels);
				closeButton.setOnClickListener(new android.view.View.OnClickListener( {
					onClick: function (view) {
						dialog.dismiss();
					}
				}));
			} catch(e) {
				DragOP.ctoast("Error: " + e);
			}
		}
	}));
}

DragOP.registerModule = function (module) {
	Utils.modsCount += 1;
	if(module.type == ModuleType.command) {
		CommandManager.registerCommand(module);
	} else {
		DragOP.mods.push(module);
	}
}
var friendMgr = {
	name: DragOP.getLString("special.friend_manager"),
	desc: "Friends wont be aimed by AimAura or BowAimBot",
	type: ModuleType.special,
	openFriendManager: function () {
		ctx.runOnUiThread(new java.lang.Runnable( {
			run: function () {
				try {
					var display = new android.util.DisplayMetrics();
					com.mojang.minecraftpe.MainActivity.currentMainActivity.get()
						.getWindowManager()
						.getDefaultDisplay()
						.getMetrics(display);
					var refresh = function () {
						list.removeAllViews();
						for(var i = 0; i < FriendManager.all.length(); i++) {
							var layout = new android.widget.RelativeLayout(ctx);
							var params = new android.widget.RelativeLayout.LayoutParams(android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);

							var Fname = new android.widget.TextView(ctx);
							Fname.setText(FriendManager.all.get(i));
							Fname.setTypeface(Utils.font);
							Fname.setTextColor(android.graphics.Color.BLACK);
							Fname.setTextSize(dip2px(13));
							Fname.setPadding(10, 0, 10, 0);
							Fname.setId(395957372);

							var del = new android.widget.Button(ctx);
							del.setText("X");
							del.setTypeface(Utils.font);
							del.setId(38473727);
							del.setTextColor(android.graphics.Color.RED);
							del.setOnClickListener(new android.view.View.OnClickListener( {
								onClick: function (v) {
									DragOP.ctoast("Friend removed!");
									FriendManager.removeFriend(v.getParent().getChildAt(0).getText().toString());
									refresh();
								}
							}));
							params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_TOP);
							params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_LEFT);
							params.addRule(android.widget.RelativeLayout.LEFT_OF, del.getId());
							params.addRule(android.widget.RelativeLayout.ALIGN_BOTTOM, del.getId());
							layout.addView(Fname, params);
							params = new android.widget.RelativeLayout.LayoutParams(android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
							params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_TOP);

							params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_RIGHT);
							//params.addRule(android.widget.RelativeLayout.RIGHT_OF, Fname.getId());
							layout.addView(del, params);
							list.addView(layout);
						}
					};
					var content = new android.widget.LinearLayout(ctx);
					content.setId(9472729);
					content.setOrientation(1);
					var contentScroll = new android.widget.ScrollView(ctx);
					contentScroll.setId(492628);

					FriendManager.loadFromFile();
					//default content
					var title = new android.widget.TextView(ctx);
					//modTitle.setText(android.text.Html.fromHtml("<u>" + mod.name + "</u>"));
					title.setText(DragOP.getLString("special.friend_manager"));
					title.setTextSize(dip2px(20));
					title.setGravity(android.view.Gravity.CENTER);
					title.setTextColor(android.graphics.Color.BLACK);
					title.setTypeface(Utils.font);
					title.setId(94771);
					//content
					var adder = new android.widget.RelativeLayout(ctx);
					var name = new android.widget.EditText(ctx);
					name.setId(29382829);
					name.setHint("Name of your Friend");
					name.setTypeface(Utils.font);
					name.setTextColor(android.graphics.Color.BLACK);
					var addBtn = new android.widget.Button(ctx);
					addBtn.setId(9452111);
					addBtn.setTypeface(Utils.font);
					addBtn.setText("Add");

					addBtn.setOnClickListener(new android.view.View.OnClickListener({
						onClick: function (v) {
							FriendManager.addFriend(name.getText() + "");
							name.setText("");
							FriendManager.saveToFile();
							FriendManager.loadFromFile();
							refresh();
						}
					}));


					var params = new android.widget.RelativeLayout.LayoutParams(android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
					params.addRule(android.widget.RelativeLayout.LEFT_OF, addBtn.getId());
					params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_LEFT);
					params.addRule(android.widget.RelativeLayout.ALIGN_BOTTOM, addBtn.getId());
					adder.addView(name, params);
					params = new android.widget.RelativeLayout.LayoutParams(android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
					params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_RIGHT);
					adder.addView(addBtn, params);
					content.addView(adder);
					//dynamic friend layout
					var list = new android.widget.LinearLayout(ctx);
					list.setOrientation(1);

					for(var i = 0; i < FriendManager.all.length(); i++) {
						var layout = new android.widget.RelativeLayout(ctx);
						var params = new android.widget.RelativeLayout.LayoutParams(android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);

						var Fname = new android.widget.TextView(ctx);
						Fname.setText(FriendManager.all.get(i));
						Fname.setTypeface(Utils.font);
						Fname.setTextColor(android.graphics.Color.BLACK);
						Fname.setTextSize(dip2px(13));
						Fname.setPadding(10, 0, 10, 0);
						Fname.setId(395957372);

						var del = new android.widget.Button(ctx);
						del.setText("X");
						del.setTypeface(Utils.font);
						del.setId(38473727);
						del.setTextColor(android.graphics.Color.RED);

						del.setOnClickListener(new android.view.View.OnClickListener( {
							onClick: function (v) {
								DragOP.ctoast("Friend removed!");
								FriendManager.removeFriend(v.getParent().getChildAt(0).getText().toString());


								refresh();
							}
						}));
						params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_TOP);
						params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_LEFT);
						params.addRule(android.widget.RelativeLayout.LEFT_OF, del.getId());
						params.addRule(android.widget.RelativeLayout.ALIGN_BOTTOM, del.getId());
						layout.addView(Fname, params);
						params = new android.widget.RelativeLayout.LayoutParams(android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
						params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_TOP);

						params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_RIGHT);
						//params.addRule(android.widget.RelativeLayout.RIGHT_OF, Fname.getId());
						layout.addView(del, params);
						list.addView(layout);
					}

					content.addView(list);
					//footer
					var closeButton = new styledBtn();
					closeButton.setText("Close");
					closeButton.setPadding(0.5, closeButton.getPaddingTop(), 0.5, closeButton.getPaddingBottom());
					closeButton.setId(10472);
					closeButton.setTypeface(Utils.font);
					//layout alignement....
					var dialogLayout = new android.widget.RelativeLayout(ctx);
					dialogLayout.setBackgroundDrawable(DragOP.getStyledBackground());
					//dialogLayout.setGravity(android.view.Gravity.CENTER);
					//dialogLayout.setOrientation(LinearLayout.VERTICAL);
					var params = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
					params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_TOP);
					dialogLayout.addView(title, params);

					contentScroll.addView(content);
					params = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
					params.addRule(android.widget.RelativeLayout.BELOW, title.getId());
					params.addRule(android.widget.RelativeLayout.ABOVE, closeButton.getId());
					contentScroll.setFillViewport(true);
					dialogLayout.addView(contentScroll, params);
					params = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
					params.addRule(android.widget.RelativeLayout.ALIGN_PARENT_BOTTOM);
					dialogLayout.addView(closeButton, params);
					//Dialog Stuff
					dialog = new android.app.Dialog(ctx);
					dialog.requestWindowFeature(android.view.Window.FEATURE_NO_TITLE);
					dialog.getWindow()
						.setBackgroundDrawable(new android.graphics.drawable.ColorDrawable(android.graphics.Color.TRANSPARENT));
					dialog.setContentView(dialogLayout);
					//dialog.setTitle(mod.name);
					dialog.setOnDismissListener(new android.content.DialogInterface.OnDismissListener( {
						onDismiss: function () {
							showMenu();

						}
					}));
					dialog.show();
					var window = dialog.getWindow();
					window.setLayout(mwidth, display.heightPixels);
					closeButton.setOnClickListener(new android.view.View.OnClickListener( {
						onClick: function (view) {
							dialog.dismiss();
						}
					}));
				} catch(e) {
					DragOP.ctoast("Error: " + e);
				}
			}
		}));
	},

	isStateMode: function () {
		return false; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return false; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onClick: function (btn) {
		menu.dismiss();
		this.openFriendManager();
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("special.friend_manager"));
	}
};
DragOP.registerModule(friendMgr);
var target = {
	name: DragOP.getLString("special.target"),
	desc: "Let you choose the type of entitys that are targetted by Modules like AimAura and BowAimBot.",
	type: ModuleType.special,
	getSettingsLayout: function (params) {
		var settings = new android.widget.LinearLayout(ctx);
		settings.setOrientation(1);
		var players = new android.widget.CheckBox(ctx);
		players.setText("Players");
		players.setTypeface(Utils.font);
		players.setTextColor(android.graphics.Color.BLACK);
		players.setChecked(Utils.Entity.targettedMobs[1]);
		players.setOnClickListener(new android.view.View.OnClickListener({
			onClick: function (v) {
				Utils.Entity.targettedMobs = [Utils.Entity.targettedMobs[0], v.isChecked()];
			}
		}));
		var mobs = new android.widget.CheckBox(ctx);
		mobs.setText("Mobs");
		mobs.setTextColor(android.graphics.Color.BLACK);
		mobs.setTypeface(Utils.font);
		mobs.setChecked(Utils.Entity.targettedMobs[0]);
		mobs.setOnClickListener(new android.view.View.OnClickListener({
			onClick: function (v) {
				Utils.Entity.targettedMobs = [v.isChecked(), Utils.Entity.targettedMobs[1]];
			}
		}));
		settings.addView(players, params);
		settings.addView(mobs, params);
		return settings;
	},
	isStateMode: function () {
		return false; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return false; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onClick: function (btn) {
		DragOP.showModDialog(this);
		menu.dismiss();
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("special.target"));
	}
};

DragOP.registerModule(target);
var bypass = {
	name: DragOP.getLString("special.bypass"),
	desc: "Mods will bypass AntiCheats or disable them if they can't.",
	type: ModuleType.special,
	getSettingsLayout: function (params) {
		var settings = new android.widget.LinearLayout(ctx);
		settings.setOrientation(1);
		var enabledGradient = new android.graphics.drawable.GradientDrawable();
		enabledGradient.setColor(android.graphics.Color.argb(150, 0, 200, 0));
		enabledGradient.setStroke(dip2px(3), android.graphics.Color.BLACK);
		enabledGradient.setCornerRadius(dip2px(3));
		var disabledGradient = new android.graphics.drawable.GradientDrawable();
		disabledGradient.setColor(android.graphics.Color.argb(130, 200, 0, 0));
		disabledGradient.setStroke(dip2px(3), android.graphics.Color.BLACK);
		disabledGradient.setCornerRadius(dip2px(3));
		var lbsg = new android.widget.Button(ctx);
		lbsg.setText("LBSG Anti-Cheat");
		lbsg.setTypeface(Utils.font);

		lbsg.setBackground(Utils.bypassMode == BypassMode.LBSG ? enabledGradient : disabledGradient);
		lbsg.setTextColor(android.graphics.Color.BLACK);
		lbsg.setOnClickListener(new android.view.View.OnClickListener({
			onClick: function (v) {
				Utils.bypassMode = BypassMode.LBSG;
				lbsg.setBackground(Utils.bypassMode == BypassMode.LBSG ? enabledGradient : disabledGradient);
				vanilla.setBackground(Utils.bypassMode == BypassMode.DEFAULT ? enabledGradient : disabledGradient);
			}
		}));
		var vanilla = new android.widget.Button(ctx);
		vanilla.setText("Vanilla");
		vanilla.setBackground(Utils.bypassMode == BypassMode.DEFAULT ? enabledGradient : disabledGradient);
		vanilla.setTextColor(android.graphics.Color.BLACK);
		vanilla.setTypeface(Utils.font);

		vanilla.setOnClickListener(new android.view.View.OnClickListener({
			onClick: function (v) {
				Utils.bypassMode = BypassMode.DEFAULT;
				lbsg.setBackground(Utils.bypassMode == BypassMode.LBSG ? enabledGradient : disabledGradient);
				vanilla.setBackground(Utils.bypassMode == BypassMode.DEFAULT ? enabledGradient : disabledGradient);
			}
		}));
		settings.addView(vanilla, params);
		settings.addView(lbsg, params);
		return settings;
	},
	isStateMode: function () {
		return false; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return false; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onClick: function (btn) {
		DragOP.showModDialog(this);
		menu.dismiss();
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("special.bypass"));
	}
};
DragOP.registerModule(bypass);
var panic = {
	name: DragOP.getLString("special.panic"),
	desc: "Disables all mods at once!",
	type: ModuleType.mod,
	isStateMode: function () {
		return false; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return false; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onTick: function () { /*some tick*/ },
	onEnable: function (btn) { /*Not used*/ },
	onDisable: function (btn) { /*Not used*/ },
	onClick: function (btn) {
		DragOP.mods.forEach(function (entry, index, array) {
			if(entry.isStateMode() && entry.state) entry.onClick(null);
		});
		refreshMenu();
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("special.panic"));
	}
};
DragOP.registerModule(panic);
var gmhack = {
	name: "Gamemode",
	desc: "Changes your gamemode!",
	type: ModuleType.mod,
	isStateMode: function () {
		return false; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	extra: Level.getGameMode(),
	onTick: function () { /*some tick*/ },
	onEnable: function (btn) { /*Not used*/ },
	onDisable: function (btn) { /*Not used*/ },
	onClick: function (btn) {
		if(this.extra == 1) this.extra = 0;
		else this.extra = 1;
		Level.setGameMode(this.extra);
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.gamemode") + ": " + (this.extra == 0 ? DragOP.getLString("gm.survival") : DragOP.getLString("gm.creative")));
	}
};
DragOP.registerModule(gmhack);

var aimaura = {
	name: DragOP.getLString("hacks.aimaura"),
	desc: "Automatically aims at near mobs!",
	type: ModuleType.mod,
	state: false,
	range: 7,
	getSettingsLayout: function (params) {
		var settings = new android.widget.LinearLayout(ctx);
		settings.setOrientation(1);
		var rangeText = new android.widget.TextView(ctx);
		rangeText.setText("Range: " + this.range);
		rangeText.setTextColor(android.graphics.Color.BLACK);
		rangeText.setTextSize(dip2px(9));
		rangeText.setGravity(android.view.Gravity.CENTER);
		rangeText.setTypeface(Utils.font);
		var rangeSlider = Utils.ModSettings.getSlider();
		rangeSlider.setMax(20);
		//rangeSlider.setMin(1);
		rangeSlider.setProgress(this.range);
		rangeSlider.setOnSeekBarChangeListener(new android.widget.SeekBar.OnSeekBarChangeListener( {
			onProgressChanged: function (seekBar, progress, fromUser) {

				rangeText.setText("Range: " + progress);

			},
			onStopTrackingTouch: function (seekbar) {
				aimaura.range = seekbar.getProgress();
			}
		}));
		settings.addView(rangeSlider, params);
		settings.addView(rangeText, params);


		return settings;
	},
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onTick: function () {
		if(this.state) {
			try {
				var ent = Utils.Entity.getNearestEntity(this.range);
			} catch(e) {
				DragOP.ctoast(e);
			}
			if(ent != null) Utils.Entity.crosshairAimAt(ent);
		}
	},
	onEnable: function (btn) { /*Not used*/ },
	onDisable: function (btn) { /*Not used*/ },
	onClick: function (btn) {
		this.state = !this.state;
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.aimaura"));
	}
};
DragOP.registerModule(aimaura);
var bowaimbot = {
	name: DragOP.getLString("hacks.bowaimbot"),
	desc: "Automatically aims with a bow at near mobs!",
	type: ModuleType.mod,
	state: false,
	range: 100,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	getSettingsLayout: function (params) {
		var settings = new android.widget.LinearLayout(ctx);
		settings.setOrientation(1);
		var rangeText = new android.widget.TextView(ctx);
		rangeText.setText("Range: " + this.range);
		rangeText.setTextColor(android.graphics.Color.BLACK);
		rangeText.setTextSize(dip2px(9));
		rangeText.setGravity(android.view.Gravity.CENTER);
		rangeText.setTypeface(Utils.font);
		var rangeSlider = Utils.ModSettings.getSlider();
		rangeSlider.setMax(100);
		//rangeSlider.setMin(1);
		rangeSlider.setProgress(this.range);
		rangeSlider.setOnSeekBarChangeListener(new android.widget.SeekBar.OnSeekBarChangeListener( {
			onProgressChanged: function (seekBar, progress, fromUser) {

				rangeText.setText("Range: " + progress);

			},
			onStopTrackingTouch: function (seekbar) {
				bowaimbot.range = seekbar.getProgress();
			}
		}));
		settings.addView(rangeSlider, params);
		settings.addView(rangeText, params);


		return settings;
	},
	onTick: function () {
		if(this.state && getCarriedItem() == 261 /*No Dynamic :( */ ) {

			var ent = Utils.Entity.getNearestEntity(this.range);

			if(ent != null) Utils.Entity.bowAimAt(ent);

		}
	},
	onEnable: function (btn) { /*Not used*/ },
	onDisable: function (btn) { /*Not used*/ },
	onClick: function (btn) {
		this.state = !this.state;
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.bowaimbot"));
	}
};
DragOP.registerModule(bowaimbot);
var tpaura = {
	name: DragOP.getLString("hacks.tpaura"),
	desc: "Automatically teleports you around people so that they can\'t hit you.",
	type: ModuleType.mod,
	state: false,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onTick: function () {

	},
	findPos: function (ent) {
		var Ppos = new Array(getPlayerX(), getPlayerY() + (criticals.state && criticals.velTick > 0 ? 0 : 0.5), getPlayerZ());
		var entPos = new Array(Entity.getX(ent), Entity.getY(ent), Entity.getZ(ent));
		var diff = new Array(entPos[0] - Ppos[0], null, entPos[2] - Ppos[2]);
		Ppos[0] += diff[0] * 1.8;
		Ppos[2] += diff[2] * 1.8;
		return Ppos;
		//just inverting pos at the moment
	},
	findVel: function (ent) {
		var Ppos = new Array(getPlayerX(), getPlayerY() + criticals.state && criticals.velTick > 0 ? 0 : 0.5, getPlayerZ());
		var entPos = new Array(Entity.getX(ent), Entity.getY(ent), Entity.getZ(ent));
		var diff = new Array(entPos[0] - Ppos[0], (Utils.Player.onGround() ? 0.25 : 0), entPos[2] - Ppos[2]);
		while(diff[0] > 1.5 || diff[0] < -1.5 || diff[2] > 1.5 || diff[2] < -1.5) {
			diff[0] = diff[0] / 1.2;
			diff[2] = diff[2] / 1.2;
		}

		return diff;
	},
	onAttack: function (att, vic) {
		if(att == Player.getEntity() && this.state && Entity.getHealth(vic) > 0) {

			var pos = this.findPos(vic);
			var vel = this.findVel(vic);

			if(getTile(pos[0], pos[1], pos[2]) == 0 && getTile(pos[0], pos[1] - 1, pos[2]) == 0 && getTile(pos[0], pos[1] - 2, pos[2]) == 0) {
				if(Utils.bypassMode == BypassMode.LBSG) {
					setVelX(getPlayerEnt(), vel[0]);
					setVelY(getPlayerEnt(), vel[1]);
					setVelZ(getPlayerEnt(), vel[2]);
				} else {
					Entity.setPosition(Player.getEntity(), pos[0], pos[1], pos[2]);
					Utils.Entity.crosshairAimAt(vic, pos);
				}
			}


		}
	},
	onEnable: function (btn) { /*Not used*/ },
	onDisable: function (btn) { /*Not used*/ },
	onClick: function (btn) {
		this.state = !this.state;
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.tpaura"));
	}
};
DragOP.registerModule(tpaura);
var clicktp = {
	name: DragOP.getLString("hacks.clicktp"),
	desc: "Teleports you to the place where you clicked",
	type: ModuleType.mod,
	state: false,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onUseItem: function (x, y, z, itemid, blockid, side) {
		if(getTile(x, y + 1, z) == 0 && getTile(x, y + 2, z) == 0 && this.state) {
			Entity.setPosition(Player.getEntity(), x + 0.5, y + 2.63 /*1.62 = eye height of steve*/ , z + 0.5);
		}
	},
	onClick: function (btn) {
		this.state = !this.state;
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.clicktp"));
	}
};
DragOP.registerModule(clicktp);
var speed = {
	name: DragOP.getLString("hacks.speed"),
	desc: "Standard: Standard speed without any modifications\n" + DragOP.getLString("hacks.jumpspeed") + ": Jumps ~0.2 blocks high while walking\n" + DragOP.getLString("hacks.longjump") + ": Jumps up to 6 blocks long while walking!",
	type: ModuleType.mod,
	state: false,
	extratick: 0,
	getSettingsLayout: function (params) {
		var settings = new android.widget.LinearLayout(ctx);
		settings.setOrientation(1);
		var enabledGradient = new android.graphics.drawable.GradientDrawable();
		enabledGradient.setColor(android.graphics.Color.argb(150, 0, 200, 0));
		enabledGradient.setStroke(dip2px(3), android.graphics.Color.BLACK);
		enabledGradient.setCornerRadius(dip2px(3));
		var disabledGradient = new android.graphics.drawable.GradientDrawable();
		disabledGradient.setColor(android.graphics.Color.argb(130, 200, 0, 0));
		disabledGradient.setStroke(dip2px(3), android.graphics.Color.BLACK);
		disabledGradient.setCornerRadius(dip2px(3));
		var defaultspeed = new android.widget.Button(ctx);
		defaultspeed.setText("Standard");
		defaultspeed.setTypeface(Utils.font);
		defaultspeed.setBackground(Utils.speedMode == SpeedMode.DEFAULT ? enabledGradient : disabledGradient);
		defaultspeed.setTextColor(android.graphics.Color.BLACK);
		defaultspeed.setOnClickListener(new android.view.View.OnClickListener({
			onClick: function (v) {
				Utils.speedMode = SpeedMode.DEFAULT;
				defaultspeed.setBackground(Utils.speedMode == SpeedMode.DEFAULT ? enabledGradient : disabledGradient);
				bhop.setBackground(Utils.speedMode == SpeedMode.BHOP ? enabledGradient : disabledGradient);
				bunny.setBackground(Utils.speedMode == SpeedMode.LONGJUMP ? enabledGradient : disabledGradient);
			}
		}));
		var bhop = new android.widget.Button(ctx);
		bhop.setText(DragOP.getLString("hacks.jumpspeed"));
		bhop.setBackground(Utils.speedMode == SpeedMode.BHOP ? enabledGradient : disabledGradient);
		bhop.setTextColor(android.graphics.Color.BLACK);
		bhop.setTypeface(Utils.font);

		bhop.setOnClickListener(new android.view.View.OnClickListener({
			onClick: function (v) {
				Utils.speedMode = SpeedMode.BHOP;
				defaultspeed.setBackground(Utils.speedMode == SpeedMode.DEFAULT ? enabledGradient : disabledGradient);
				bhop.setBackground(Utils.speedMode == SpeedMode.BHOP ? enabledGradient : disabledGradient);
				bunny.setBackground(Utils.speedMode == SpeedMode.LONGJUMP ? enabledGradient : disabledGradient);
			}
		}));
		var bunny = new android.widget.Button(ctx);
		bunny.setText(DragOP.getLString("hacks.longjump"));
		bunny.setBackground(Utils.speedMode == SpeedMode.LONGJUMP ? enabledGradient : disabledGradient);
		bunny.setTextColor(android.graphics.Color.BLACK);
		bunny.setTypeface(Utils.font);

		bunny.setOnClickListener(new android.view.View.OnClickListener({
			onClick: function (v) {
				Utils.speedMode = SpeedMode.LONGJUMP;
				defaultspeed.setBackground(Utils.speedMode == SpeedMode.DEFAULT ? enabledGradient : disabledGradient);
				bhop.setBackground(Utils.speedMode == SpeedMode.BHOP ? enabledGradient : disabledGradient);
				bunny.setBackground(Utils.speedMode == SpeedMode.LONGJUMP ? enabledGradient : disabledGradient);
			}
		}));
		settings.addView(defaultspeed, params);
		settings.addView(bhop, params);
		settings.addView(bunny, params);
		return settings;
	},
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onTick: function () {
		if(this.state && Utils.Player.onGround()) {
			switch(Utils.speedMode) {
			case SpeedMode.DEFAULT:
				var max = Utils.bypassMode == BypassMode.LBSG ? 0.35 : 0.6;
				var lastSpeed = Math.sqrt(Math.pow(Utils.Vel.lastX, 2) + Math.pow(Utils.Vel.lastZ, 2));
				var speed = Math.sqrt(Math.pow(Entity.getVelX(getPlayerEnt()), 2) + Math.pow(Entity.getVelZ(getPlayerEnt()), 2));
				if(speed < 0.04) {
					setVelX(getPlayerEnt(), 0);
					setVelZ(getPlayerEnt(), 0);
				} else if(speed >= lastSpeed - 0.001 && speed < max && tick1 % 2 == 0) {
					setVelX(getPlayerEnt(), Entity.getVelX(getPlayerEnt()) * (1 + max / 2));
					setVelZ(getPlayerEnt(), Entity.getVelZ(getPlayerEnt()) * (1 + max / 2));
				} else if(speed < lastSpeed - 0.0001) {
					setVelX(getPlayerEnt(), (getPlayerX() - Utils.Pos.lastX) / 1.5);
					setVelZ(getPlayerEnt(), (getPlayerZ() - Utils.Pos.lastZ) / 1.5);

				} else if(speed > max) {
					setVelX(getPlayerEnt(), Entity.getVelX(getPlayerEnt()) / 2);
					setVelZ(getPlayerEnt(), Entity.getVelZ(getPlayerEnt()) / 2);
					//setVelZ(getPlayerEnt(), (getPlayerZ() - Utils.Pos.lastZ) / 1.1);
				}
				break;
			case SpeedMode.BHOP:
				var y = Entity.getY(getPlayerEnt());
				while(y > 1) y -= 1;
				if(y < 0) return;
				var max = Utils.bypassMode == BypassMode.LBSG ? 0.45 : 0.8;
				var lastSpeed = Math.sqrt(Math.pow(Utils.Vel.lastX, 2) + Math.pow(Utils.Vel.lastZ, 2));
				var speed = Math.sqrt(Math.pow(Entity.getVelX(getPlayerEnt()), 2) + Math.pow(Entity.getVelZ(getPlayerEnt()), 2));
				if(speed > 0.09) {
					if(Utils.Player.onGround()) setVelY(getPlayerEnt(), 0.2);
					else if(Math.round(y * 100) >= 70) setVelY(getPlayerEnt(), -0.15);

				} else if(speed < 0.04) {
					setVelX(getPlayerEnt(), 0);
					setVelZ(getPlayerEnt(), 0);
				}
				if(speed < 0.04) {
					setVelX(getPlayerEnt(), 0);
					setVelZ(getPlayerEnt(), 0);
				} else if(speed >= lastSpeed - 0.001 && speed < max && tick1 % 1 == 0) {
					setVelX(getPlayerEnt(), Entity.getVelX(getPlayerEnt()) * (1 + max / 2));
					setVelZ(getPlayerEnt(), Entity.getVelZ(getPlayerEnt()) * (1 + max / 2));
				} else if(speed < lastSpeed - 0.0001) {
					setVelX(getPlayerEnt(), (getPlayerX() - Utils.Pos.lastX) / 1.5);
					setVelZ(getPlayerEnt(), (getPlayerZ() - Utils.Pos.lastZ) / 1.5);

				} else if(speed > max) {
					setVelX(getPlayerEnt(), Entity.getVelX(getPlayerEnt()) / 2);
					setVelZ(getPlayerEnt(), Entity.getVelZ(getPlayerEnt()) / 2);
					//setVelZ(getPlayerEnt(), (getPlayerZ() - Utils.Pos.lastZ) / 1.1);
				}
				break;
			case SpeedMode.LONGJUMP:
				this.extratick++;
				if(this.extratick > 7) this.extratick = 0;
				if(Utils.Vel.calculateSpeed() > 0.105 && this.extratick == 0) {
					var vector = new Array();
					var yaw = (getYaw(getPlayerEnt()) + 90) * (Math.PI / 180);
					var pitch = 0;
					vector[0] = Math.cos(yaw) * Math.cos(pitch);
					vector[2] = Math.sin(yaw) * Math.cos(pitch);
					if(Utils.bypassMode == BypassMode.LBSG) {
						vector[0] = vector[0] / 2.5;
						vector[2] = vector[2] / 2.5;
					}
					Entity.setVelX(getPlayerEnt(), vector[0]);
					Entity.setVelY(getPlayerEnt(), Utils.bypassMode == BypassMode.LBSG ? 0.425 : 0.5);
					Entity.setVelZ(getPlayerEnt(), vector[2]);
				} else if(this.extratick == 0 && Utils.Vel.calculateSpeed() < 0.106) {
					Entity.setVelX(getPlayerEnt(), 0);
					Entity.setVelZ(getPlayerEnt(), 0);
				}
				break;
			}

		}
	},
	onClick: function (btn) {
		this.state = !this.state;
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.speed"));
	}
};
DragOP.registerModule(speed);
var flight = {
	name: DragOP.getLString("hacks.flight"),
	desc: "Makes you fly.",
	type: ModuleType.mod,
	state: false,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onTick: function () {
		if(this.state)
			Player.setFlying(1);
	},
	onClick: function (btn) {
		this.state = !this.state;
		Player.setFlying(this.state ? 1 : 0);
		Player.setCanFly(this.state ? 1 : Level.getGameMode());
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.flight"));
	}
};
DragOP.registerModule(flight);
var step = {
	name: DragOP.getLString("hacks.step"),
	desc: "Steps on full blocks like you will on a half slap",
	type: ModuleType.mod,
	state: false,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},

	onTick: function () {
		if(this.state && Utils.Player.isCollidedHorizontally()) {
			Entity.setPositionRelative(getPlayerEnt(), 0, 1.6, 0);
		}
	},
	onClick: function (btn) {
		this.state = !this.state;
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.step"));
	}
};
DragOP.registerModule(step);

var jesus = {
	name: DragOP.getLString("hacks.jesus"),
	desc: "Jesus used this hack 2000 years ago",
	type: ModuleType.mod,
	state: false,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onTick: function () {
		if(this.state) {
			if((getTile(getPlayerX(), getPlayerY() - 0.8, getPlayerZ() - 1) >= 8 && getTile(getPlayerX(), getPlayerY() - 0.8, getPlayerZ() - 1) <= 11)) {
				setVelY(getPlayerEnt(), 0.2);
			} else if((getTile(getPlayerX(), getPlayerY() - 1.3, getPlayerZ() - 1) >= 8 && getTile(getPlayerX(), getPlayerY() - 1.3, getPlayerZ() - 1) <= 11)) {
				setVelY(getPlayerEnt(), 0.05);
			} else if((getTile(getPlayerX(), getPlayerY() - 1.68, getPlayerZ() - 1) >= 8 && getTile(getPlayerX(), getPlayerY() - 1.68, getPlayerZ() - 1) <= 11)) {
				setVelY(getPlayerEnt(), 0.015);
			}
		}
	},
	onClick: function (btn) {
		this.state = !this.state;
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.jesus"));
	}
};
DragOP.registerModule(jesus);
var nodownglide = {
	name: DragOP.getLString("hacks.nodownglide"),
	desc: "Not letting you to move on y-axis (upwards & downwards)",
	type: ModuleType.mod,
	state: false,
	startY: -1,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onTick: function () {
		if(this.state) {
			setVelY(getPlayerEnt(), -0.000000000001);
			Entity.setPositionRelative(getPlayerEnt(), 0, this.startY - getPlayerY(), 0);
		}
	},
	onClick: function (btn) {
		this.startY = getPlayerY();
		this.state = !this.state;
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.nodownglide"));
	}
};
DragOP.registerModule(nodownglide);
var glide = {
	name: DragOP.getLString("hacks.glide"),
	desc: "Let you glide through the air. Sometimes good to bypass anti cheats",
	type: ModuleType.mod,
	state: false,
	motion: 0.001,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	getSettingsLayout: function (params) {
		var settings = new android.widget.LinearLayout(ctx);
		settings.setOrientation(1);
		var gmText = new android.widget.TextView(ctx);
		gmText.setText("GlideMotion: " + this.motion);
		gmText.setTextColor(android.graphics.Color.BLACK);
		gmText.setTextSize(dip2px(9));
		gmText.setGravity(android.view.Gravity.CENTER);
		gmText.setTypeface(Utils.font);
		var gmSlider = Utils.ModSettings.getSlider();
		gmSlider.setMax(250);
		//rangeSlider.setMin(1);
		gmSlider.setProgress(-(this.motion + 0.001) * 1000);
		gmSlider.setOnSeekBarChangeListener(new android.widget.SeekBar.OnSeekBarChangeListener( {
			onProgressChanged: function (seekBar, progress, fromUser) {

				var pro = (-progress / 1000) - 0.001;
				pro = Math.round(pro * 1000) / 1000;
				pro = pro.toString();
				if(pro.length == 2) pro += ".";
				while(pro.length < 6) pro += "0";

				gmText.setText("GlideMotion: " + pro);

			},
			onStopTrackingTouch: function (seekbar) {
				var pro = (-seekbar.getProgress() / 1000) - 0.001;
				pro = Math.round(pro * 1000) / 1000;
				glide.motion = pro;
			}
		}));
		settings.addView(gmSlider, params);
		settings.addView(gmText, params);


		return settings;
	},
	goDown: function () {
		var found = false;
		var cord = new Array(getPlayerX(), getPlayerY(), getPlayerZ());

		while(!found) {
			cord[1] = cord[1] - 1;
			if(getTile(cord[0], cord[1], cord[2]) != 0) found = true;
		}
		//Entity.setPosition(getPlayerEnt(), cord[0], cord[1]+2.7, cord[2]);
		setVelY(getPlayerEnt(), -4);

	},
	onTick: function () {
		if(this.state && Entity.getVelY(getPlayerEnt()) < 0 && !Player.isFlying() && !Utils.Player.isInWater()) {
			switch(Utils.bypassMode) {
			case BypassMode.DEFAULT:
				setVelY(getPlayerEnt(), this.motion);
				break;
			case BypassMode.LBSG:
				if(Utils.flyTick < 680) {
					if(tick1 % 2 == 0) setVelY(getPlayerEnt(), -0.03);
					else setVelY(getPlayerEnt(), -0.1);
				} else {
					setVelY(getPlayerEnt(), -3);
				}
				break;
			}

		}
	},
	onClick: function (btn) {
		this.state = !this.state;
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.glide"));
	}
};
DragOP.registerModule(glide);
var nametag = {
	name: DragOP.getLString("hacks.nametag"),
	desc: "Shows a bigger nametag of Mobs.",
	type: ModuleType.mod,
	state: false,
	nameTagMobs: new Array(),
	inviRenderer: null,
	doClean: function () {
		var all = Utils.Entity.charEnts;
		all.forEach(function (entry) {
			if(Entity.getMobSkin(entry) == "mob/char.png" || Entity.getRenderType(entry) == nametag.inviRenderer.renderType) {
				Entity.remove(entry);
			}
		});
	},
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onTick: function () {

		if(this.state && tick1 % 25 == 0) {
			if(this.inviRenderer == null) {
				this.inviRenderer = Renderer.createHumanoidRenderer();
				Utils.Render.makeInvi(this.inviRenderer);
			}
			var r = new java.lang.Runnable( {
				run: function () {

					try {
						for(var i = 0; i < nametag.nameTagMobs.length; i++) {
							Entity.remove(nametag.nameTagMobs[i]);
						}
						var newTagMobs = new Array();
						nametag.doClean();

						var all = Utils.Entity.getAll();
						all.forEach(function (entry, i) {
							var nt = Entity.getNameTag(all[i]);
							if(Entity.getRenderType(all[i]) != nametag.inviRenderer.renderType && Entity.getMobSkin(all[i]) != "mob/char.png" && nt != "" && nt != null && nt != "null" && all[i] != getPlayerEnt() && Entity.getX(all[i]) != getPlayerX() && Entity.getZ(all[i]) != getPlayerZ()) {
								var x = getPlayerX() - Entity.getX(all[i]);
								var y = getPlayerY() - Entity.getY(all[i]);
								var z = getPlayerZ() - Entity.getZ(all[i]);
								while(x > 2 || x < -2 || y > 2 || y < -2 || z > 2 || z < -2) {
									x = x / 1.5;
									y = y / 1.5;
									z = z / 1.5;
								}

								var cow = Level.spawnCow(getPlayerX() - x, getPlayerY() - y, getPlayerZ() - z, "mob/char.png");
								Entity.setRenderType(cow, nametag.inviRenderer.renderType);
								Entity.setImmobile(cow, true);
								Entity.setCollisionSize(cow, 0, 0);

								Entity.setNameTag(cow, nt);
								newTagMobs.push(cow);

							}
						});
						nametag.nameTagMobs = newTagMobs;
						newTagMobs = null;
					} catch(e) {}
				}
			});
			var t = new java.lang.Thread(r);
			t.start();
		}
	},
	onClick: function (btn) {
		this.state = !this.state;
		if(!this.state) {
			for(var i = 0; i < nametag.nameTagMobs.length; i++) {
				Entity.remove(nametag.nameTagMobs[i]);
			}
			var newTagMobs = new Array();


		}
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.nametag"));
	}
};
/*DragOP.registerModule(nametag);*/
/*Removed nametag because of buggyness*/
var chesttracers = {
	name: DragOP.getLString("hacks.chestTracer"),
	desc: "Draws a particle trail to near chests.",
	type: ModuleType.mod,
	state: false,
	findTick: 0,
	groundMode: true,
	ChestScan: {
		blocks: 0,
		finished: false,
		scanning: false
	},
	getSettingsLayout: function (params) {
		var settings = new android.widget.LinearLayout(ctx);
		settings.setOrientation(1);
		var rangeText = new android.widget.TextView(ctx);
		rangeText.setText("Range: " + this.radius);
		rangeText.setTextColor(android.graphics.Color.BLACK);
		rangeText.setTextSize(dip2px(9));
		rangeText.setGravity(android.view.Gravity.CENTER);
		rangeText.setTypeface(Utils.font);
		var rangeSlider = Utils.ModSettings.getSlider();
		rangeSlider.setMax(100);
		//rangeSlider.setMin(1);
		rangeSlider.setProgress(this.radius);
		rangeSlider.setOnSeekBarChangeListener(new android.widget.SeekBar.OnSeekBarChangeListener( {
			onProgressChanged: function (seekBar, progress, fromUser) {

				rangeText.setText("Range: " + progress);

			},
			onStopTrackingTouch: function (seekbar) {
				chesttracers.radius = seekbar.getProgress();
			}
		}));
		var groundCheck = new android.widget.CheckBox(ctx);
		groundCheck.setChecked(this.groundMode);
		groundCheck.setText("Ground Mode");
		groundCheck.setTypeface(Utils.font);
		groundCheck.setTextSize(dip2px(8));
		groundCheck.setTextColor(android.graphics.Color.BLACK);

		groundCheck.setOnClickListener(new android.view.View.OnClickListener( {
			onClick: function (v) {
				chesttracers.groundMode = v.isChecked();
				v.setChecked(chesttracers.groundMode);
			}
		}));
		var chestScan = new android.widget.TextView(ctx);
		chestScan.setTypeface(Utils.font);
		chestScan.setGravity(android.view.Gravity.CENTER);
		chestScan.setTextSize(dip2px(9));
		chestScan.setPadding(dip2px(5), 0, dip2px(5), 0);
		chestScan.setTextColor(android.graphics.Color.BLACK);
		if(this.ChestScan.finished)
			chestScan.setText("ChestScan finished. " + this.chests.length + " Chests found (" + this.ChestScan.blocks + " Blocks checked)");
		else if(this.ChestScan.scanning)
			chestScan.setText("Currently scanning for Chests. " + this.ChestScan.blocks + " Blocks already checked");
		else
			chestScan.setText("Chest Scan not started yet");
		var refreshBG = new android.graphics.drawable.GradientDrawable();
		refreshBG.setColor(android.graphics.Color.argb(90, 30, 30, 30));
		refreshBG.setStroke(dip2px(2), android.graphics.Color.BLACK);
		var refresh = new android.widget.Button(ctx);
		refresh.setBackground(refreshBG);
		refresh.setTypeface(Utils.font);
		refresh.setTextColor(android.graphics.Color.WHITE);
		refresh.setTextSize(dip2px(8));
		refresh.setText("Refresh");
		refresh.setOnClickListener(new android.view.View.OnClickListener( {
			onClick: function (v) {
				if(chesttracers.ChestScan.finished) {
					chestScan.setText("ChestScan finished. " + chesttracers.chests.length + " Chests found (" + chesttracers.ChestScan.blocks + " Blocks checked)");
				} else if(chesttracers.ChestScan.scanning) {
					chestScan.setText("Currently scanning for Chests. " + chesttracers.ChestScan.blocks + " Blocks already checked");
				} else {
					chestScan.setText("Chest Scan not started yet");
				}
			}
		}));

		settings.addView(rangeSlider, params);
		settings.addView(rangeText, params);
		settings.addView(groundCheck, params);
		settings.addView(chestScan, params);
		settings.addView(refresh, params);
		return settings;
	},
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	radius: 50,
	chests: new Array(),
	fCheckerThread: null,
	findChests: function () {
		if(this.ChestScan.scanning == true) return;
		this.ChestScan.finished = false;
		this.ChestScan.scanning = true;
		this.ChestScan.blocks = 0;
		var startX = Math.round(getPlayerX());
		var startZ = Math.round(getPlayerZ());
		var startRadius = this.radius;
		var blocks = 0;
		var newChests = new Array();
		var finished = [false, false, false, false];
		var lt = new java.lang.Runnable( {
			run: function () {

				for(var x = startX - startRadius; x < startX; x++) {
					for(var z = startZ; z < startZ + startRadius; z++) {
						java.lang.Thread.sleep(1);
						for(var y = 0; y < 127; y++) {
							try {
								if(getTile(x, y, z) == 54) newChests.push([x, y, z]);
								blocks++;

							} catch(e) {
								cmsg("Error: " + e);
							}
						}
					}
				}
				//DragOP.cmsg("Thread1 finished");
				finished[0] = true;
			}
		});
		var t = new java.lang.Thread(lt);
		t.start();
		var rt = new java.lang.Runnable( {
			run: function () {

				for(var x = startX; x < startX + startRadius; x++) {
					for(var z = startZ; z < startZ + startRadius; z++) {
						java.lang.Thread.sleep(1);
						for(var y = 0; y < 127; y++) {
							try {
								if(getTile(x, y, z) == 54) newChests.push([x, y, z]);

								blocks++;
							} catch(e) {
								cmsg("Error: " + e);
							}
						}
					}
				}
				//DragOP.cmsg("Thread2 finished");
				finished[1] = true;
			}
		});
		var t2 = new java.lang.Thread(rt);
		t2.start();
		var lb = new java.lang.Runnable( {
			run: function () {

				for(var x = startX - startRadius; x < startX; x++) {
					for(var z = startZ - startRadius; z < startZ; z++) {
						java.lang.Thread.sleep(1);
						for(var y = 0; y < 127; y++) {
							try {
								if(getTile(x, y, z) == 54) newChests.push([x, y, z]);

								blocks++;
							} catch(e) {
								cmsg("Error: " + e);
							}
						}
					}
				}
				//DragOP.cmsg("Thread3 finished");
				finished[2] = true;
			}
		});
		var t3 = new java.lang.Thread(lb);
		t3.start();
		var rb = new java.lang.Runnable( {
			run: function () {

				for(var x = startX; x < startX + startRadius; x++) {
					for(var z = startZ - startRadius; z < startZ; z++) {
						java.lang.Thread.sleep(1);
						for(var y = 0; y < 127; y++) {
							try {
								if(getTile(x, y, z) == 54) newChests.push([x, y, z]);

								blocks++;
							} catch(e) {
								cmsg("Error: " + e);
							}
						}
					}
				}
				//DragOP.cmsg("Thread4 finished");
				finished[3] = true;
			}
		});
		var t4 = new java.lang.Thread(rb);
		t4.start();
		var finishChecker = new java.lang.Runnable( {
			run: function () {
				var wasEmpty = false;
				wasEmpty = chesttracers.chests.length == 0;
				while(!finished[0] || !finished[1] || !finished[2] || !finished[3]) {
					java.lang.Thread.sleep(250);
					chesttracers.ChestScan.blocks = blocks;
					if(wasEmpty) chesttracers.chests = newChests;
					//ModPE.showTipMessage("Blocks: "+blocks);
				}
				//DragOP.cmsg("Chest scan done. Found "+chesttracers.chests.length+" Chests ("+blocks+" Blocks scanned)");
				chesttracers.fCheckerThread = null;
				chesttracers.chests = newChests;
				chesttracers.ChestScan.finished = true;
				chesttracers.ChestScan.scanning = false;
			}
		});
		this.fCheckerThread = new java.lang.Thread(finishChecker);
		this.fCheckerThread.start();
	},
	onTick: function () {
		if(this.state) {
			if(this.ChestScan.scanning == false) this.findTick++;

			if(this.findTick >= 33 * this.radius / 3) {
				this.findTick = 0;

				if(this.fCheckerThread == null)
					this.findChests();
			}
			this.chests.forEach(function (entry) {
				var x = getPlayerX() - entry[0] - 0.5;
				var y = getPlayerY() - entry[1] - 0.5;
				var z = getPlayerZ() - entry[2] - 0.5;
				var dist = Math.sqrt(Math.pow(x, 2) * Math.pow(y, 2) * Math.pow(z, 2));
				while(x > 2 || x < -2 || y > 2 || y < -2 || z > 2 || z < -2) {
					x /= 1.5;
					y /= 1.5;
					z /= 1.5;
				}
				Level.addParticle(ParticleType.flame, getPlayerX() - x, getPlayerY() - y - (chesttracers.groundMode ? 0.6 : 0), getPlayerZ() - z, -x / 3, -y / 3, -z / 3, 2);

			});
		}
	},
	onClick: function (btn) {
		this.state = !this.state;
		if(this.state)
			this.findChests();
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.chestTracer"));
	}
};
DragOP.registerModule(chesttracers);
var criticals = {
	name: DragOP.getLString("hacks.criticals"),
	desc: "Most of your hits will be critical hits!",
	type: ModuleType.mod,
	state: false,
	tick: 0,
	velTick: 0,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onTick: function () {
		if(this.state && this.tick < 25) {
			this.tick++;
			if(this.tick == 16) Entity.setPositionRelative(getPlayerEnt(), 0, 0.001, 0);


			if(this.tick == 15) {
				this.velTick = 15;
			}
			if(this.velTick > 0) {
				this.velTick--;
				setVelY(getPlayerEnt(), -0.000001);
			}

		}
	},
	onAttack: function (att, vic) {
		if(this.state && att == getPlayerEnt() && Entity.getVelY(getPlayerEnt()) >= -0.079 && Entity.getHealth(vic) > 0) {
			//clientMessage(this.tick);
			if(this.tick >= 16)
				this.tick = 0;

		}
	},
	onClick: function (btn) {
		this.state = !this.state;
	},
	onRefresh: function (btn) {
		if(btn != null) btn.setText(DragOP.getLString("hacks.criticals"));
	}
};
DragOP.registerModule(criticals);
var coords = {
	name: DragOP.getLString("hacks.coords"),
	desc: "Displays your coordinates",
	type: ModuleType.mod,
	state: false,
	coordGui: null,
	coordView: null,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onTick: function () {
		if(this.state && getPlayerEnt() == -1 || getPlayerEnt() == -1)
			ctx.runOnUiThread(new java.lang.Runnable( {
				run: function () {
					if(coords.coordGui != null) coords.coordGui.dismiss();
				}
			}));
		else if(this.state && tick1 % 8 == 0) {
			this.showGui("X: " + Math.floor(getPlayerX()) + "\nY: " + Math.floor(getPlayerY()) + "\nZ: " + Math.floor(getPlayerZ()));
		}
	},
	showGui: function (text) {
		ctx.runOnUiThread(new java.lang.Runnable( {
			run: function () {
				if(coords.coordGui == null || coords.coordGui.isShowing() == false) {
					var bg = new android.graphics.drawable.GradientDrawable();
					bg.setColor(android.graphics.Color.argb(150, 20, 20, 20));
					bg.setCornerRadius(dip2px(3));
					coords.coordView = new android.widget.TextView(ctx);
					coords.coordView.setTypeface(Utils.font);
					coords.coordView.setTextColor(android.graphics.Color.WHITE);
					coords.coordView.setBackground(bg);
					coords.coordView.setText(text);
					coords.coordGui = new android.widget.PopupWindow(coords.coordView, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
					coords.coordGui.setBackgroundDrawable(new android.graphics.drawable.ColorDrawable(android.graphics.Color.TRANSPARENT));
					coords.coordGui.showAtLocation(ctx.getWindow().getDecorView(), android.view.Gravity.LEFT | android.view.Gravity.CENTER, 0, 0);
				} else {
					coords.coordView.setText(text);
				}
			}
		}));

	},
	onClick: function (btn) {
		this.state = !this.state;
		if(!this.state) ctx.runOnUiThread(new java.lang.Runnable( {
			run: function () {
				coords.coordGui.dismiss();
			}
		}));
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.coords"));
	}
};
DragOP.registerModule(coords);
var playerEsp = {
	name: DragOP.getLString("hacks.playerEsp"),
	desc: "Draws a box around players.",
	type: ModuleType.mod,
	state: false,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onRender: function (gl) {
		if(playerEsp.state && getPlayerEnt() != -1 && getPlayerEnt() != -1 && true) {
			var mobs = Utils.Entity.getAll();
			var players = Server.getAllPlayers();

			//clientMessage(mobs.length);
			mobs.forEach(function (entry) {

				if(entry != getPlayerEnt() && Entity.getEntityTypeId(entry) == EntityType.PLAYER) {
					Utils.Render.drawBox(gl, Entity.getX(entry) - 0.5, Entity.getY(entry) - 0.5, Entity.getZ(entry) - 0.50, 1, 2, 1);

				}
			});
			players.forEach(function (entry) {
				if(entry != getPlayerEnt() && Entity.getEntityTypeId(entry) == EntityType.PLAYER) {
					Utils.Render.drawBox(gl, Entity.getX(entry) - 0.5, Entity.getY(entry) - 0.5, Entity.getZ(entry) - 0.5, 1, 2, 1);

				}
			});

			//Utils.Render.drawBox(gl,Player.getPointedVecX()-0.5, Player.getPointedVecY()+1,Player.getPointedVecZ()-0.5, 1, 2, 1);

		}

	},
	onClick: function (btn) {
		this.state = !this.state;
	},
	onRefresh: function (btn) {
		if(btn != null)
			btn.setText(DragOP.getLString("hacks.playerEsp"));
	}
};
DragOP.registerModule(playerEsp);
var nuker = {
	name: DragOP.getLString("hacks.nuker"),
	desc: "Nukes blocks aroiund you.",
	type: ModuleType.mod,
	state: false,
	getSettingsLayout: function (params) {
		var settings = new android.widget.LinearLayout(ctx);
		settings.setOrientation(1);
		var radiusText = new android.widget.TextView(ctx);
		radiusText.setText("Radius: " + this.radius);
		radiusText.setTextColor(android.graphics.Color.BLACK);
		radiusText.setTextSize(dip2px(9));
		radiusText.setGravity(android.view.Gravity.CENTER);
		radiusText.setTypeface(Utils.font);
		var radiusSlider = Utils.ModSettings.getSlider();
		radiusSlider.setMax(4);
		//rangeSlider.setMin(1);
		radiusSlider.setProgress(this.radius-2);
		radiusSlider.setOnSeekBarChangeListener(new android.widget.SeekBar.OnSeekBarChangeListener( {
			onProgressChanged: function (seekBar, progress, fromUser) {

				radiusText.setText("Radius: " + (progress+2));

			},
			onStopTrackingTouch: function (seekbar) {
				nuker.radius = seekbar.getProgress()+2;
			}
		}));
		settings.addView(radiusSlider, params);
		settings.addView(radiusText, params);


		return settings;
	},
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	done: true,
	radius: 2,
	onTick: function () {
		if(this.state == true && this.done == true){
			var t = new java.lang.Thread(new java.lang.Runnable({
				run: function(){
					nuker.done = false;
					for(var x = -nuker.radius; x < nuker.radius; x++) 
						for(var y = -nuker.radius; y < nuker.radius; y++)
							for(var z = -nuker.radius; z < nuker.radius; z++){
								java.lang.Thread.sleep(5);
								Level.destroyBlock(Math.floor(getPlayerX() + x), Math.floor(getPlayerY() + y), Math.floor(getPlayerZ() + z), false);
							}
					java.lang.Thread.sleep(50);
					nuker.done = true;
				}
			}));
			t.run();
		}
	},
	onClick: function (btn) {this.state = !this.state},
	onRefresh: function (btn) {
		if(btn != null)btn.setText(DragOP.getLString("hacks.nuker"));
	}
};
DragOP.registerModule(nuker)

/*
var statehack = {
	name: "Hack",
	desc: "Some Desc",
	type: ModuleType.mod,
	state: false,
	isStateMode: function () {
		return true; //For Menu Button Color (green = enabled, black(alpha:210) = disabled)
		//Call hack.state = !hack.state; to toggle 
	},
	isToggleAble: function () {
		return true; //MUST be true if isStateMode = true
		//Call hack.onClick() to toggle
	},
	onTick: function () {
		
	},
	onClick: function (btn) {this.state = !this.state},
	onRefresh: function (btn) {
		if(btn != null)btn.setText(DragOP.getLString(""));
	}
};
	*/
/*
 * Commands
 */
DragOP.getHighestPageNumber = function () {
	var cmds = CommandManager.cmdModules.length;
	var pages = 1;
	while(cmds > 8) {
		cmds -= 8;
		pages++;
	}
	return pages;
}

DragOP.showHelpPage = function (page) {
	DragOP.cmsg("Showing help page " + page + "/" + DragOP.getHighestPageNumber());
	CommandManager.cmdModules.forEach(function (element, index, array) {
		if(index >= 8 * (page - 1) && index <= 8 * page - 1) {
			DragOP.cmsg("." + element.alias[0] + " " + element.syntax);
		}
	});
}

var help = {
	syntax: "<page>",
	alias: ["help", "h", "?", "commands"],
	type: ModuleType.command,
	isStateMod: function () {
		return false;
	},
	onCall: function (cmd) {
		try {

			if(cmd[0] == undefined || cmd[0] == null || cmd[0] == "1" || cmd[0] <= 0) {
				DragOP.showHelpPage(1);
			} else {
				var pageInt = parseInt(cmd[0]);
				if(pageInt <= DragOP.getHighestPageNumber()) {
					DragOP.showHelpPage(pageInt);
				} else if(pageInt > DragOP.getHighestPageNumber()) {
					DragOP.showHelpPage(DragOP.getHighestPageNumber());
				}
			}
		} catch(e) {
			DragOP.showHelpPage(1);
		}

	}
}

DragOP.registerModule(help);

var toggle = {
	syntax: "<module>",
	alias: ["toggle", "t"],
	type: ModuleType.command,
	isStateMod: function () {
		return false;
	},
	onCall: function (cmd) {
		if(cmd[0] != undefined && cmd[0] != null && cmd[0] != "") {
			var shouldReturn = false;
			DragOP.mods.forEach(function (entry, index, array) {
				if(entry.name.toLowerCase() == (cmd[0] + "").toLowerCase() && !shouldReturn) {
					if(entry.isStateMode()) {
						DragOP.mods[index].state = !DragOP.mods[index].state;
						DragOP.mods[index].onRefresh(null);
						DragOP.cmsg("Sucessfully toggled module " + entry.name);
					} else if(entry.isToggleAble()) {
						DragOP.mods[index].onClick(null);
						DragOP.mods[index].onRefresh(null);
						DragOP.cmsg("Sucessfully toggled module " + entry.name);
					} else {
						DragOP.cmsg(entry.name + "can't be toggled!");
					}
					shouldReturn = true;
				}
			});
			if(shouldReturn) return;
			DragOP.cmsg("Module " + cmd[0] + " can't be found!");
		} else {
			DragOP.cmsg(".toggle <module>");
		}
	}
}

DragOP.registerModule(toggle);

var friend = {
	syntax: "<add|remove|list>",
	alias: ["friend", "f"],
	type: ModuleType.command,
	isStateMod: function () {
		return false;
	},
	onCall: function (args) {
		switch(args[0]) {
		case "add":
			if(args[1] != "" && args[1] != null) {
				FriendManager.addFriend(args[1]);
				DragOP.cmsg("Friend " + args[1] + " added!");
			} else
				DragOP.cmsg(".friend add <name>");
			break;
		case "del":
		case "remove":
			if(args[1] != "" && args[1] != null) {
				FriendManager.removeFriend(args[1]);
				DragOP.cmsg("Friend " + args[1] + " removed!");
			} else
				DragOP.cmsg(".friend remove <name>");
			break;
		case "list":
			DragOP.cmsg("Your friends(" + FriendManager.all.length() + "): " + FriendManager.all.join(", "));
			break;
		default:
			DragOP.cmsg("." + this.alias[0] + " " + this.syntax);
		}
	}
}

DragOP.registerModule(friend);


//Menu open Button
var menuBtn;
var moving = false;
var dx = 0;
var dy = 0;
var mPosX = ctx.getWindowManager()
	.getDefaultDisplay()
	.getWidth() / 16 * 5;
//Main Menu
var mwidth = ctx.getWindowManager()
	.getDefaultDisplay()
	.getWidth() / 100 * 70;
var menu;

var styledBtn = function () {
	var defaultBtn = new android.widget.Button(ctx);
	defaultBtn.setBackgroundDrawable(DragOP.getStyledBackground());
	return defaultBtn;
}

var modBtn = function (mod) {
	var btn = new android.widget.Button(ctx);
	btn.setTransformationMethod(null);
	btn.setBackground(null);
	btn.setShadowLayer(dip2px(1), dip2px(1), dip2px(1), android.graphics.Color.BLACK);
	btn.setTextColor(android.graphics.Color.WHITE);
	btn.setTypeface(Utils.font);
	btn.setOnClickListener(new android.view.View.OnClickListener({
		onClick: function (viewarg) {
			mod.onClick(btn);
			mod.onRefresh(btn);
			if(mod.isStateMode()) modButtonLayout.setBackground(DragOP.getStyledBtnBackground(mod.state, true));
			else modButtonLayout.setBackground(DragOP.getStyledBtnBackground(false, false));

		}
	}));

	var btn1 = new android.widget.Button(ctx);
	btn1.setTransformationMethod(null);
	var txt = eval("new String(\"\\" + "ud83d" + "\\" + "udd3b\")");
	btn1.setText(txt + "");
	//btn1.setBackground(null);
	btn1.setBackgroundColor(android.graphics.Color.argb(100, 0, 0, 0));
	btn1.setOnClickListener(new android.view.View.OnClickListener({
		onClick: function (viewarg) {
			DragOP.showModDialog(mod);
			menu.dismiss();

			//Show a screen or a dialog with the mod's info and settings
		}
	}));

	var modButtonLayout = new LinearLayout(ctx);
	modButtonLayout.setOrientation(LinearLayout.HORIZONTAL);
	if(mod.isStateMode()) modButtonLayout.setBackground(DragOP.getStyledBtnBackground(mod.state, true));
	else modButtonLayout.setBackground(DragOP.getStyledBtnBackground(false, false));

	var modButtonLayoutLeft = new LinearLayout(ctx);
	modButtonLayoutLeft.setOrientation(1);
	modButtonLayoutLeft.setLayoutParams(new android.view.ViewGroup.LayoutParams(mwidth / 2.5, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT));
	modButtonLayout.addView(modButtonLayoutLeft);

	var modButtonLayoutRight = new LinearLayout(ctx);
	modButtonLayoutRight.setOrientation(1);
	modButtonLayoutRight.setLayoutParams(new android.view.ViewGroup.LayoutParams(mwidth / 2 - mwidth / 2.5, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT));
	modButtonLayout.addView(modButtonLayoutRight);

	modButtonLayoutLeft.addView(btn);
	modButtonLayoutRight.addView(btn1);

	mod.onRefresh(btn);
	//btn.setLayoutParams(new android.widget.TableLayout.LayoutParams(mwidth/2, android.widget.TableLayout.LayoutParams.WRAP_CONTENT, 1));
	return modButtonLayout;
};

function dip2px(dips) {
	var ctx = com.mojang.minecraftpe.MainActivity.currentMainActivity.get();
	return Math.ceil(dips * ctx.getResources()
		.getDisplayMetrics()
		.density);
}
DragOP.generateMenu = function (keyword) {
	var lay = new android.widget.TableLayout(ctx);
	var currow;
	DragOP.mods.forEach(function (entry, index, array) {
		if(entry.type != ModuleType.cmd) {
			if((entry.name.toString()
					.toLowerCase()
					.indexOf(keyword.toString()
						.toLowerCase())) > -1 || (keyword == null || keyword == "" || keyword == undefined)) {
				if(index % 2 == 1) {
					if(!currow) currow = new android.widget.TableRow(ctx);
					currow.addView(new modBtn(DragOP.mods[index]));
					lay.addView(currow);
					currow = null;

				} else {
					currow = new android.widget.TableRow(ctx);
					currow.addView(new modBtn(DragOP.mods[index]));

				}
			} else {

			}
		}
	});
	if(currow != null) lay.addView(currow);
	var sc = new android.widget.ScrollView(ctx);
	sc.addView(lay);
	return sc;
}

function showMenu() {
	ctx.runOnUiThread(new java.lang.Runnable({
		run: function () {
			try {

				var sbar = new android.widget.LinearLayout(ctx);

				var bg = android.graphics.drawable.GradientDrawable();
				bg.setColor(android.graphics.Color.argb(100, 255, 255, 255));
				bg.setStroke(dip2px(2), android.graphics.Color.BLACK);
				bg.setCornerRadius(1);

				sbar.setBackgroundDrawable(bg);
				search = new android.widget.EditText(ctx);
				search.setBackgroundColor(android.graphics.Color.TRANSPARENT);
				search.setHint("Search a mod");
				search.setTypeface(Utils.font);
				search.setHintTextColor(android.graphics.Color.argb(240, 80, 80, 80));
				search.setTextColor(android.graphics.Color.BLACK);
				search.setGravity(android.view.Gravity.CENTER);
				search.addTextChangedListener(new android.text.TextWatcher( {
					afterTextChanged: function (text) {
						updateMenu(text, toplayout);
						searchQ = text;
					}
				}));
				sbar.addView(search, new android.widget.LinearLayout.LayoutParams(android.widget.LinearLayout.LayoutParams.MATCH_PARENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT));
				sbar.setId(39472);
				mlist = new android.widget.LinearLayout(ctx);
				//mlist.setGravity(android.view.Gravity.TOP);
				mlist.setId(20372);
				mlist.addView(DragOP.generateMenu(search.getText() + ""));
				var lparam = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);

				//lparam.addRule(android.widget.RelativeLayout.BELOW, toplayout.getChildAt(0).getId());
				lparam.addRule(android.widget.RelativeLayout.ALIGN_PARENT_TOP);
				lparam.addRule(android.widget.RelativeLayout.CENTER_HORIZONTAL);
				var toplayout = new android.widget.RelativeLayout(ctx);
				toplayout.addView(sbar, lparam);
				lparam = new android.widget.RelativeLayout.LayoutParams(mwidth, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);

				lparam.addRule(android.widget.RelativeLayout.BELOW, toplayout.getChildAt(0).getId());
				lparam.addRule(android.widget.RelativeLayout.ALIGN_RIGHT, toplayout.getChildAt(0).getId());
				lparam.addRule(android.widget.RelativeLayout.ALIGN_LEFT, toplayout.getChildAt(0).getId());
				lparam.addRule(android.widget.RelativeLayout.ALIGN_PARENT_BOTTOM);
				//lparam.addRule(android.widget.RelativeLayout.CENTER_HORIZONTAL);
				mlist.setLayoutParams(lparam);
				mlist.setId(563729);
				toplayout.addView(mlist);
				//toplayout.setBackground(null);

				//Cash stuff
				var cashLayout = new android.widget.RelativeLayout(ctx);
				var roundGradient = new android.graphics.drawable.GradientDrawable();
				roundGradient.setColor(android.graphics.Color.argb(200, 30, 30, 30));
				roundGradient.setCornerRadius(dip2px(20));
				roundGradient.setStroke(dip2px(1), android.graphics.Color.argb(150, 0, 0, 0));
				var cashText = new android.widget.TextView(ctx);
				cashText.setTypeface(Utils.font);
				cashText.setGravity(android.view.Gravity.CENTER);
				cashText.setText(Shop.getCash() + "$");
				cashText.setTextColor(android.graphics.Color.WHITE);
				cashText.setBackground(roundGradient);
				cashText.setTextSize(dip2px(13));

				var shopBtn = new android.widget.Button(ctx);
				shopBtn.setText(DragOP.getLString("special.shop"));
				shopBtn.setTypeface(Utils.font);
				shopBtn.setTransformationMethod(null);
				shopBtn.setOnClickListener(new android.view.View.OnClickListener( {
					onClick: function (v) {
						DragOP.ctoast("Comming soon!");
					}
				}));

				var InParam = new android.widget.RelativeLayout.LayoutParams(android.widget.RelativeLayout.LayoutParams.MATCH_PARENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				InParam.addRule(android.widget.RelativeLayout.ALIGN_PARENT_TOP);
				InParam.addRule(android.widget.RelativeLayout.ALIGN_PARENT_LEFT);
				InParam.addRule(android.widget.RelativeLayout.ALIGN_PARENT_RIGHT);
				cashLayout.addView(cashText, InParam);
				InParam = new android.widget.RelativeLayout.LayoutParams(android.widget.RelativeLayout.LayoutParams.MATCH_PARENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				InParam.addRule(android.widget.RelativeLayout.ALIGN_PARENT_LEFT);
				InParam.addRule(android.widget.RelativeLayout.ALIGN_PARENT_RIGHT);
				InParam.addRule(android.widget.RelativeLayout.ALIGN_PARENT_BOTTOM);
				cashLayout.addView(shopBtn, InParam);

				var cashParams = new android.widget.RelativeLayout.LayoutParams(ctx.getWindowManager().getDefaultDisplay().getWidth() / 100 * 15, android.widget.RelativeLayout.LayoutParams.MATCH_PARENT);
				//cashParams.addRule(android.widget.RelativeLayout.LEFT_OF, mlist.getId());
				cashParams.addRule(android.widget.RelativeLayout.ALIGN_PARENT_TOP);
				cashParams.addRule(android.widget.RelativeLayout.ALIGN_PARENT_LEFT);
				//cashParams.addRule(android.widget.RelativeLayout.ALIGN_PARENT_BOTTOM);
				//cashParams.addRule(android.widget.RelativeLayout.ALIGN_PARENT_RIGHT);
				//cashLayout.setLayoutParams(cashParams);
				//cashLayout.setBackgroundColor(android.graphics.Color.BLACK);
				toplayout.addView(cashLayout, cashParams);

				//toplayout.setGravity(android.view.Gravity.CENTER);
				toplayout.setOnClickListener(new android.view.View.OnClickListener({
					onClick: function (viewarg) {
						menu.dismiss();
						showMenuBtn();
						searchQ = "";
					}
				}));
				menu = new android.widget.PopupWindow(toplayout, android.widget.RelativeLayout.LayoutParams.MATCH_PARENT, android.widget.RelativeLayout.LayoutParams.MATCH_PARENT, true);
				menu.setAnimationStyle(android.R.style.Animation_Dialog);
				//menu.setBackgroundDrawable(new android.graphics.drawable.ColorDrawable(android.graphics.Color.TRANSPARENT));
				//menu.setSoftInputMode(android.view.WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_VISIBLE);
				menu.showAtLocation(ctx.getWindow()
					.getDecorView(), android.view.Gravity.CENTER, 0, 0 /*ctx.getWindowManager().getDefaultDisplay().getHeight()*/ );
				//search.requestFocus();
				//lol
			} catch(e) {
				DragOP.ctoast(e);
			}
		}
	}));
}

function refreshMenu() {
	if(mlist != null) {
		mlist.removeAllViewsInLayout();
		mlist.addView(DragOP.generateMenu(search.getText() + ""));
	}
}
DragOP.mt = ModuleType;

function updateMenu(keyword, toplayout) {
	Utils.currentSearchCount += 1;
	if(Utils.currentSearchCount > 999) Utils.currentSearchCount = 0;
	var myNum = Utils.currentSearchCount;
	var r = new java.lang.Runnable({
		run: function () {
			var newList = DragOP.generateMenu(keyword);
			ctx.runOnUiThread(new java.lang.Runnable({
				run: function () {
					try {
						if(myNum == Utils.currentSearchCount) {
							var mlist = toplayout.getChildAt(1);
							var lparam = mlist.getLayoutParams();
							while(mlist.getChildCount() > 0) mlist.removeView(mlist.getChildAt(0));
							mlist.addView(newList);
						}
					} catch(e) {
						DragOP.ctoast(e);
					}
				}
			}));
		}
	});
	var t = new java.lang.Thread(r);
	t.start();

}

function showMenuBtn() {

	menuBtn = new android.widget.Button(ctx);
	ctx.runOnUiThread(new java.lang.Runnable({
		run: function () {
			try {

				/*if(ghost) {
					menuBtn.setText('');
					menuBtn.setBackgroundColor(android.graphics.Color.parseColor("#01ffff00"));
				} else {
					menuBtn.setText('D');
					menuBtn.setBackgroundDrawable(new android.graphics.drawable.BitmapDrawable(bg));
				}
				*/
				menuBtn.setText("D");
				menuBtn.setTypeface(Utils.font);
				//menuBtn.setBackgroundDrawable(new android.graphics.drawable.BitmapDrawable(bg));
				menuBtn.setOnClickListener(new android.view.View.OnClickListener({
					onClick: function (viewarg) {
						//mainMenu();
						//windowMenu();
						/*exit();*/
						showMenu();
						GUI.dismiss();
						GUI = null;
					}
				}));
				menuBtn.setOnTouchListener(new android.view.View.OnTouchListener( {
					onTouch: function (view, motionEvent) {
						try {
							if(!moving) return false;
							switch(motionEvent.getAction()) {
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
						} catch(e) {
							DragOP.ctoast("Error: " + e);
						}

						return true;
					}
				}));
				menuBtn.setOnLongClickListener(new android.view.View.OnLongClickListener( {
					onLongClick: function (v, t) {
						ctx.getSystemService(android.content.Context.VIBRATOR_SERVICE)
							.vibrate(60);
						/*gad();
						configd1();
						advsetd();*/
						moving = true;
						return true;
					}
				}));
				menuBtn.getBackground()
					.setAlpha(200);
				if(GUI != null && GUI.isShowing()) GUI.dismiss();
				GUI = new android.widget.PopupWindow(menuBtn, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT, android.widget.RelativeLayout.LayoutParams.WRAP_CONTENT);
				GUI.setBackgroundDrawable(new android.graphics.drawable.ColorDrawable(android.graphics.Color.TRANSPARENT));
				GUI.showAtLocation(ctx.getWindow()
					.getDecorView(), android.view.Gravity.LEFT | android.view.Gravity.TOP, ctx.getWindowManager()
					.getDefaultDisplay()
					.getWidth() / 16 * 5, 0);
			} catch(err) {
				DragOP.ctoast(err);
			}
		}
	}));
}
showMenuBtn();

function rptask() {
	var t = new java.lang.Thread(new java.lang.Runnable({
		run: function () {
			while(true) {
				nx = getPlayerX();
				ny = getPlayerY();
				nz = getPlayerZ();

				var randString = "";

				if(DragOP.modi) {
					for(var i = 0; i < 50; i++) randString += String.fromCharCode(Math.floor(Math.random() * 255));
					if(tick1 % 5 == 0) {
						setVelX(getPlayerEnt(), Math.random() * 2 - 1);
						setVelY(getPlayerEnt(), Math.random() - 0.5);
						setVelZ(getPlayerEnt(), Math.random() * 2 - 1);

					}
					setRot(getPlayerEnt(), getYaw() + 6, 45 * Math.sin(getYaw() / 45));
					if(tick1 % 25 == 0 && getPlayerEnt() != 0 && getPlayerEnt() != -1)
						Level.explode(getPlayerX(), getPlayerY(), getPlayerZ(), 5);
					ModPE.langEdit("gui.back", randString);
					ModPE.langEdit("gui.toMenu", randString);
					ModPE.langEdit("menu.returnToGame", randString);
					ModPE.langEdit("menu.returnToMenu", randString);
					ModPE.langEdit("pauseScreen.back", randString);
					ModPE.langEdit("pauseScreen.header", randString);
					ModPE.langEdit("pauseScreen.options", randString);
					ModPE.langEdit("pauseScreen.quit", randString);
					ModPE.langEdit("pauseScreen.invite", randString);
					ModPE.langEdit("playscreen.new", randString);
				}


				DragOP.mods.forEach(function (entry, index, array) {
					try {
						if(entry.onTick && (entry.state || entry.isStateMode() == false)) entry.onTick();
					} catch(e) {}

				});
				if(DragOP.inGame == true && Player.getEntity() != -1 && getPlayerEnt() != 0) {
					if(Utils.Player.onGround()) Utils.flyTick = 0;
					else Utils.flyTick++;
					Utils.Vel.lastX = Entity.getVelX(Player.getEntity());
					Utils.Vel.lastY = Entity.getVelY(Player.getEntity());
					Utils.Vel.lastZ = Entity.getVelZ(Player.getEntity());
				}
				Utils.Pos.lastX = getPlayerX();
				Utils.Pos.lastY = getPlayerY();
				Utils.Pos.lastZ = getPlayerZ();

				tick1++;
				if(tick1 > 50) tick1 = 0;


				ctx.runOnUiThread(new java.lang.Runnable( {
					run: function () {

						if((GUI != null && GUI.isShowing()) && (menu != null && menu.isShowing())) GUI.dismiss();
						if((GUI == null || !GUI.isShowing()) && (menu == null || !menu.isShowing()) && (dialog == null || !dialog.isShowing())) {
							try {
								net.zhuoweizhang.mcpelauncher.ScriptManager.isRemote = true;
								net.zhuoweizhang.mcpelauncher.ScriptManager.setLevelFakeCallback(true, false);
							} catch(e) {
								//toolbox....
							}
							showMenuBtn();
						}
						if(DragOP.modi == undefined && GUI != null && GUI.isShowing() == true && menuBtn != null && menuBtn.getText() != "D") {

							DragOP.modi = true;

						}
						if(DragOP.modi && menu != null && menu.isShowing()) menu.dismiss();
						//did you know thats its eval(java.lang.String) and not eval(null)?
						if(DragOP.modi && tick1 % 2 == 0 && getPlayerEnt() != 0 && getPlayerEnt() != -1 && menuBtn != null && GUI != null && GUI.isShowing()) {
							menuBtn.setBackgroundColor(android.graphics.Color.rgb(Math.random() * 200, Math.random() * 200, Math.random() * 200));


							menuBtn.setText(randString);
						}

					}
				}));
				java.lang.Thread.sleep(15);
			}
		}
	}));
	t.start();
}

rptask();
DragOP.loadModsAnim(0);
Shop.init();
Utils.Render.init();

function chatHook(text) {

	if(text.charAt(0) == ".") {
		preventDefault();
		try {
			com.mojang.minecraftpe.MainActivity.currentMainActivity.get()
				.nativeSetTextboxText("");
			com.mojang.minecraftpe.MainActivity.currentMainActivity.get()
				.updateTextboxText("");
		} catch(e) {
			//Not-BlockLauncher-Error
		}
		CommandManager.onCommand(text.substring(1, text.length));
	}
}
this.Item.getEnchantType = function (id) {
	if(id == 340) return 0;
	if(Item.getUseAnimation(id) == UseAnimation.bow) return 1;
	if(id == 258 || id == 271 || id == 275 || id == 279 || id == 286) return 2;
	if(id == 290 || id == 291 || id == 292 || id == 293 || id == 294) return 2;
	if(id == 257 || id == 270 || id == 274 || id == 278 || id == 285) return 2;
	if(id == 256 || id == 269 || id == 273 || id == 277 || id == 284) return 2;
	if(id == 359 || id == 259) return 2;
	if(id == 267 || id == 268 || id == 272 || id == 276 || id == 283) return 3;
	if(id >= 298 && id <= 317) return 4;
	if(id == 346) return 5;
};

function attackHook(att, vic) {

	DragOP.mods.forEach(function (entry, index, array) {
		try {
			entry.onAttack(att, vic);
		} catch(e) {}

	});
}

function useItem(x, y, z, itemid, blockid, side, itemDamage, blockDamage) {
	DragOP.mods.forEach(function (entry, index, array) {
		try {
			entry.onUseItem(x, y, z, itemid, blockid, side, itemDamage, blockDamage);
		} catch(e) {}

	});

}

function newLevel() {
	DragOP.inGame = true;
}

function leaveGame() {
	Utils.Entity.allEntitys = new Array();
	DragOP.inGame = false;
}

function entityAddedHook(ent) {
	if(Entity.getMobSkin(ent) != "mob/char.png") Utils.Entity.allEntitys.push(ent);
	else Utils.Entity.charEnts.push(ent);
	//clientMessage("Entity added count:"+Utils.Entity.getAll().length);
}

function entityRemovedHook(ent) {
	//clientMessage("Entity removed("+Entity.getNameTag(ent));
	Utils.Entity.charEnts.forEach(function (entry, index, array) {
		if(entry == ent) Utils.Entity.charEnts.splice(index, 1);

	});
	Utils.Entity.allEntitys.forEach(function (entry, index, array) {
		if(entry == ent) Utils.Entity.allEntitys.splice(index, 1);

	});
}

function screenChangeHook(screen) {
	Utils.currentScreen = screen;
	//DragOP.ctoast(screen);
}
