# Legacy mod
Use the following chrome extension for these mods: [voilentmonkey](https://violentmonkey.github.io/)
<details> 
  <summary>Legacy mod</summary>

```js
// ==UserScript==
// @name        Legacy Mod
// @namespace   Grifs-scripts
// @match       *://*shellshock.io/*
// @grant       GM_xmlhttpRequest
// @version     1.0
// @author      Grif
// @description Return the legacy default weapons
// ==/UserScript==

let oldLog = console.log;
const patch_legacy = () => {
  let x = [3000, 3100, 3400, 3600, 3800,4000, 4200];
  x.forEach((i) => {
    item = extern.catalog.findItemById(i);
    if (!item.item_data.meshName.includes("_Legacy")) {
      item.item_data.meshName += "_Legacy";
      console.log(`Patched ${item.name}`, item);
    }
  })
}

/*Audio shit here*/
let audio = new AudioContext;
let sounds = {};
const sound_list = [
  'cluck9mm_fire','cluck9mm_insert_mag','cluck9mm_remove_mag',
  'csg1_fire','csg1_pull_action','csg1_release_action',
  'dozenGauge_close','dozenGauge_fire','dozenGauge_load','dozenGauge_open',
  'eggk47_dry_fire','eggk47_fire','eggk47_full_cycle','eggk47_insert_mag','eggk47_remove_mag',
  'm24_bolt_close','m24_bolt_open','m24_fire',
  'rpegg_fire','rpegg_load','rpegg_rocket_fly','rpegg_rocket_hit','rpegg_rocket_poof',
  'smg_cycle','smg_fire','grenade','grenade_pin','grenade_beep', 'ammo'
];

const patch_sounds = () => {
  // window.current_sounds = BAWK.sounds; // old sfx
  try {
    sound_list.forEach((sfx) => {
      if (BAWK.sounds[sfx] == undefined && BAWK.sounds['gun_'+sfx]) {sfx = "gun_" + sfx;} // why did this need changing :/
      if (BAWK.sounds[sfx]) {
          BAWK.sounds[sfx].buffer = sounds[sfx.replace('gun_', '')]; // is this what it means to be a shitty js developer?
          BAWK.sounds[sfx].start = 0;
          BAWK.sounds[sfx].end = BAWK.sounds[sfx].buffer.duration;
      }
    })
    // window.old_sounds = BAWK.sounds; // new sfx. future proofing incase i want to beable to toggle between them (never going to happen)
  } catch (err) { console.log(`Error loading sounds: ${err}`) }
}

console.log = function () {
  if (typeof arguments[0] == 'string' && arguments[0]?.includes("sounds loaded")) {
    patch_legacy();
    patch_sounds();
  }
  return oldLog.apply(this, arguments);
}
sound_list.forEach((sound) => {
  let sfx;
  try {
    GM_xmlhttpRequest({
      method: 'GET',
      url: `https://github.com/Grifmin/Shell/blob/main/sounds/${sound}.mp3?raw=true`,
      responseType: "arraybuffer",
      onload: async function (data) {
        let buffer = data.response;
        sfx = await audio.decodeAudioData(buffer);
        sounds[sound] = sfx;
      }
    })
  } catch (err) {
    console.log(`Error fetching ${sound}:`, err);
  }
})
```
</details>


<details> 
  <summary>Map search</summary>

```js
// ==UserScript==
// @name        Map search
// @namespace   Map-search
// @match       *://*shellshock.io/*
// @inject-into page
// @version     1.0
// @author      Grif, Jayvan
// @description Allows you to search maps (wow)
// ==/UserScript==

/*update template*/
let loop = setInterval(function() {
  if (document.getElementById('create-private-game-template')) {
    let src = document.getElementById('create-private-game-template').innerHTML.replace(/(<div id="private_maps")/gm,
      `<input id="map_name_inp" :value="mapName" placeholder="Map Name" @change="onMapInpChange(\$event)" v-on:keyup="onMapNameKeyUp(\$event)" class="ss_select" style="margin-left: 5%;">\n$1`
    );
    document.getElementById('create-private-game-template').innerHTML = src;
    clearInterval(loop);
  }
},1);

/*replacing functions*/
let check = setInterval(function() {
  if (window.comp_create_private_game_popup) {
    let main = window.comp_create_private_game_popup;
    main.props.push('mapName');
    let func = main.methods;
    func.onMapInpChange = function (event) {
      event.target.value = extern.filterUnicode(event.target.value);
      event.target.value = extern.fixStringWidth(event.target.value);
      event.target.value = event.target.value.substring(0, Math.max(...vueData.maps.map(x => x.name.length)));
    }

    func.onMapNameKeyUp = function(event) {
      this.mapName = event.target.value;
      this.boxedit = true;
    }

    func.old_selectMapForPickedGameType = func.selectMapForPickedGameType;
    func.selectMapForPickedGameType = function(dir) {
      this.arrowed = true;
      this.old_selectMapForPickedGameType(dir)
    }

    let modified = func.handleKeydown.toString().replace(/(preventDefault\(\);)/gm, `$1
      if (!this.mapName || this.arrowed) {this.mapName = '', this.arrowed = false}
      if (keyPressed == 'backspace' && e.ctrlKey) this.mapName = '';
      if (!this.boxedit) {
        let k = keyPressed.replace('key', '');
        if (k == 'backspace') return this.mapName = this.mapName.substring(0, Math.max(0, this.mapName.length-1));
        if (e.target?.id != 'map_name_inp') {
          // console.log('mapname', keyPressed);
          this.mapName += k.length == 1? k : '';
          if (k == 'space' && this.mapName.length > 0) this.mapName += ' ';
        }
        this.mapName = this.mapName.substring(0, Math.max(...vueData.maps.map(x => x.name.length)));
        if (!'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 '.includes(k)) return
      }
      this.boxedit = false;
    `).replace(/let idx = vueApp\.maps\.findIndex\(el => el\.name\.toLowerCase\(\)\.startsWith\(keyPressed\.replace\('key', ''\)\)\);/gm, `
      let idx = [
        vueApp.maps.findIndex(map => map.name.toLowerCase().startsWith(this.mapName.toLowerCase()) ),
        vueApp.maps.findIndex(map => map.filename.startsWith(this.mapName.toLowerCase()) ),
        vueApp.maps.findIndex(map => (this.mapName.length > 1 && map.name.toLowerCase().includes(this.mapName.toLowerCase())) )
      ].find(i => i > -1) || -1
    `).replace(/handleKeydown\([A-z]+\) {/gm, '');
    func.handleKeydown = new Function('e', modified.substring(0, modified.length-1));
    clearInterval(check);
  }
},1);

```
</details>
