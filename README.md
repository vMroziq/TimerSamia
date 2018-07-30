# TimerSamia

function Boss(id, card) {
    this.ID = id;
    this.name = bossNames[0];
    this.color = 100;

    this.startTime = respawnTimes[0];
    this.time = this.startTime;

    this.enabled = false;

    this.timerSpan = $(card).find(".timer");
    this.nameSpan = $(card).find(".bossName");


    this.toggle = function () {
        this.enabled = !this.enabled;
    };

    this.reset = function () {
        this.enabled = false;
        this.time = this.startTime;
        this.timerSpan.text(this.convert());
        this.nameSpan.text(this.name);
    };

    this.run = function () {
        if (this.enabled) {
            this.color = (this.time / this.startTime) * 100;
            if (this.time > 0) {
                this.time--;
                this.timerSpan.text(this.convert());
                let hsltext = "hsl(" + this.color + ",75%,50%)";
                this.timerSpan.css("color", hsltext)
            } else {
                this.time = this.startTime;
            }
            if (this.time === 30) {
                this.notify();
            }
        }
    };

    this.convert = function () {
        let mins = Math.floor(this.time / 60);
        let secs = this.time % 60;

        if (mins < 10) {
            mins = "0" + mins;
        }
        if (secs < 10) {
            secs = "0" + secs;
        }
        return mins + ":" + secs
    };

    this.notify = function () {
        if (allowNotifiaiotns) {
            let options = {
                icon: 'static/boss_images/' + this.name + ".png"
            };
            let notification = new Notification(this.name + " CH" + (this.ID + 1) + " za 30 sekund!", options);
            let audio = new Audio("static/notification.mp3");
            audio.play();
        }
    }
}

let bossList = [];
let toggleButtons = [];
let resetButtons = [];
let respawnTimes = [7 * 60 + 30, 12 * 60 + 30, 26 * 60 + 30, 46 * 60 + 30, undefined, 10 * 60, 15 * 60, 30 * 60, 49 * 60 + 30];
let bossNames = ["Mylfid", "Olimpus", "Veryhtus", "Temani", undefined, "Magiczny Metin", "Metin Lodu", "Metin Spustoszenia", "Metin Asherod"];
let allowNotifiaiotns = false;

for (let i = 0; i < $(".boss").length; i++) {
    let current = $(".boss")[i];
    let boss = new Boss(i, current);
    boss.reset();
    bossList.push(boss);

    let onButton = $(current).find(".btnon");
    toggleButtons.push(onButton);

    let resetButton = $(current).find(".btnreset");
    resetButtons.push(resetButton);
}

for (let i = 0; i < toggleButtons.length; i++) {
    toggleButtons[i].click(function () {
        bossList[i].toggle()
    });
    resetButtons[i].click(function () {
        bossList[i].reset()
    });
}

$("#bossControl").change(function () {
    for (let i = 0; i < bossList.length; i++) {
        bossList[i].startTime = respawnTimes[this.selectedIndex];
        bossList[i].name = bossNames[this.selectedIndex];
        bossList[i].reset();
    }
});

$(".btnresetall").click(function () {
    for (let i = 0; i < bossList.length; i++) {
        bossList[i].reset();
    }
});


if (Notification.permission !== "denied" && Notification.permission !== "granted") {
    Notification.requestPermission(function (permission) {
        if (permission === "granted") {
            allowNotifiaiotns = true;
        }
    });
} else if (Notification.permission === "granted") {
    allowNotifiaiotns = true;
}


function tick() {
    for (let i = 0; i < bossList.length; i++) {
        bossList[i].run();
    }

}

window.onbeforeunload = function () {
    for (let i = 0; i < bossList.length; i++) {
        if (bossList[i].enabled === true) {
            return 1
        }
    }
};


setInterval(tick, 1000);
