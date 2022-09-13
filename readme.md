# Legacy mod
Use the following chrome extension for this mod: [voilentmonkey](https://violentmonkey.github.io/)
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
  'smg_cycle','smg_fire',
];

const patch_sounds = () => {
  sound_list.forEach((sfx) => {
    BAWK.sounds[sfx].buffer = sounds[sfx];
    BAWK.sounds[sfx].start = 0;
    BAWK.sounds[sfx].end = BAWK.sounds[sfx].buffer.duration
  })
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
      url: `https://github.com/Grifmin/Shell/blob/main/${sound}.mp3?raw=true`,
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
