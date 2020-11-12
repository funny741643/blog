# 中介者模式

## 作用

解除对象与对象的紧耦合关系，所有的相关对象都通过中介者对象来通信

## 泡泡堂

```javascript
var Player = function (name, teamColor) {
    this.name = name;
    this.state = "alive";
    this.teamColor = teamColor;
};
Player.prototype.win = function () {
    console.log(this.name + "won");
};
Player.prototype.lose = function () {
    console.log(this.name + "lost");
};
Player.prototype.die = function () {
    this.state = "dead";
    // 委托给中介者来执行
    playerDirector.ReceiveMessage("playerDead", this);
};
Player.prototype.remove = function () {
    // 委托给中介者来执行
    playerDirector.ReceiveMessage("removePlayer", this);
};
Player.prototype.changeTeam = function (color) {
    // 委托给中介者来执行
    playerDirector.ReceiveMessage("changeTeam", this, color);
};

var playerFactory = function (name, teamColor) {
    var newPlayer = new Player(name, teamColor);
    playerDirector.ReceiveMessage("addPlayer", newPlayer);
    return newPlayer;
};

// 中介者,负责增添玩家，移除玩家，玩家换队，消息通知
var playerDirector = (function () {
    var players = {},
        operations = {};
    operations.addPlayer = function (player) {
        var teamColor = player.teamColor;
        players[teamColor] = players[teamColor] || [];
        players[teamColor].push(player);
    };
    operations.removePlayer = function (player) {
        var teamColor = player.teamColor;
        teamPlayers = players[teamColor] || [];
        for (var i = teamPlayers.length - 1; i >= 0; i--) {
            if (teamPlayers[i] == player) {
                teamPlayers.splice(i, 1);
            }
        }
    };
    operations.changeTeam = function (player, newTeamColor) {
        operations.removePlayer(player);
        player.teamColor = newTeamColor;
        operations.addPlayer(player);
    };
    operations.playerDead = function (player) {
        var teamColor = player.teamColor,
            teamPlayers = players[teamColor];

        var all_dead = true;
        for (var i = 0; i < teamPlayers.length; i++) {
            if (teamPlayers[i].state == "alive") return (all_dead = false);
        }
        if (all_dead == true) {
            for (var i = 0; i < teamPlayers.length; i++) {
                teamPlayers[i].lose();
            }
            for (var color in players) {
                if (color !== teamColor) {
                    var teamPlayers = players[color];
                    for (var i = 0, player; (player = teamPlayers[i++]); ) {
                        player.win();
                    }
                }
            }
        }
    };

    var ReceiveMessage = function () {
        var message = Array.prototype.shift.call(arguments);
        operations[message].apply(this, arguments);
    };
    return {
        ReceiveMessage: ReceiveMessage,
    };
})();

var player1 = playerFactory("皮蛋", "red");
var player2 = playerFactory("狗蛋", "red");
var player3 = playerFactory("牛蛋", "red");
var player4 = playerFactory("蛋蛋", "red");
var player5 = playerFactory("小涛", "blue");
var player6 = playerFactory("小凡", "blue");
var player7 = playerFactory("小超", "blue");
var player8 = playerFactory("小洋", "blue");

player1.remove()
player2.remove()
player3.changeTeam('blue')
// player3.die()
player4.die()
```

