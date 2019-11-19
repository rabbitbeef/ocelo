<template>
  <div>
    <table border="solid">
      <tr v-for="(y,yindex) in board" :key="yindex">
        <td v-for="(x,xindex) in y" :key="xindex">
          <div @click="onClick(yindex,xindex)">
            <oce-box v-bind:Status="board[yindex][xindex]"></oce-box>
          </div>
        </td>
      </tr>
    </table>
    手番：{{turn}}
    <br />
    ターン:{{turns_i}}
    <br />
    <br />
    white:{{sum["white"]}}
    <br />
    black:{{sum["black"]}}
    <br />
    <button v-if="game_end" @click="resetGame">リセット</button>
  </div>
</template>

<script>
//振り返り。turnとturns_iはどっちかでよかった。1なら白。0なら黒など。

import oceloBox from "./ocelo-box.vue";

export default {
  props: {
    N: {
      type: Number,
      default: 8
    }
  },

  data: function() {
    return {

      board: Array,
      sum: {
        white: 2,
        black: 2
      },
      clear: false,
      turn: "black",
      turns: ["black", "white"],
      turns_i: 0,
      //白か黒が物理的に石を置ける範囲。
      //ターン開始ごとにこの中から置ける場所をチェック。
      //置いた後に置ける場所を更新。
      //そうするとどこに置けるかのチェックがO(N^2)で済むはず。
      put_able:[],
      put_able_first: [
          [5,5],
          [2,5],
          [5,2],
        [2, 2],
        [3, 2],
        [2, 3],
        [4, 2],
        [2, 4],
        [3, 5],
        [5, 3],
        [5, 4],
        [4, 5]
      ],
      //置いた場合の結果一覧
      put_results: [],
      game_end: {
        type: Boolean,
        default: false
      }
    };
  },

  mounted: function() {
    //盤面の初期化
    this.resetGame();

    //置ける場所を調べる
    this.put_results = this.putAbleColorCheck(this.put_able, this.turn);
  },

  methods: {
    resetGame: function() {
      this.put_able = this.put_able_first;
      this.put_results= [];
      this.sum["white"] = 2;
      this.sum["black"] = 2;
      this.turn = "black";
      this.turns_i = 0;
      this.game_end = false;
      this.board = this.resetBoard(this.N, this.N);

    },
    //盤面のリセット。横N、縦Mになる。
    resetBoard: function(N, M) {
      if (N < 5 || M < 5) {
        return console.error();
      }

      let board = new Array(M);
      for (let i = 0; i < M; i++) {
        board[i] = new Array(N);
        for (let j = 0; j < N; j++) {
          if ((i == 3 && j == 4) || (i == 4 && j == 3)) {
            board[i][j] = "black";
          } else if ((i == 3 && j == 3) || (i == 4 && j == 4)) {
            board[i][j] = "white";
          } else {
            board[i][j] = "blank";
          }
        }
      }

      return board;
    },

    //デバック用。一手で全部黒になる。
    resetBoard2: function(N, M) {
      if (N < 5 || M < 5) {
        return console.error();
      }

      let board = new Array(M);
      for (let i = 0; i < M; i++) {
        board[i] = new Array(N);
        for (let j = 0; j < N; j++) {
          if ((i == 3 && j == 4) || (i == 4 && j == 3 || (i == 4 && j == 4))) {
            board[i][j] = "black";
          } else if (i == 3 && j == 3) {
            board[i][j] = "white";
          } else {
            board[i][j] = "blank";
          }
        }
      }

      return board;
    },

    //石を置けるかチェックする。
    //置けるならば結果を返す。
    //TODO:Nとボードに依存しているのでリファクタリングするなら外す。
    putStoneCheck: function(y, x, turn) {
      let result = {
        x: x,
        y: y,
        line: []
      };
      //調べる方向ベクトル
      let direction = [
        [1, 0],
        [1, 1],
        [0, 1],
        [-1, 1],
        [-1, 0],
        [-1, -1],
        [0, -1],
        [1, -1]
      ];

      let check = false;

      for (let i = 0; i < direction.length; i++) {
        //パラメタ。
        let r = 1;

        //同じ方向に違う色の石があったかどうかチェック
        let dif = false;

        while (true) {
          const dy = direction[i][0] * r;
          const dx = direction[i][1] * r;

          const Yr = y + dy;
          const Xr = x + dx;

          //現在地が盤外に到着したら、または空白だったらその方向にはオセロ無し
          if (
            Yr >= this.N ||
            Xr >= this.N ||
            Yr < 0 ||
            Xr < 0 ||
            this.board[Yr][Xr] === "blank"
          ) {
            break;
          }

          //手番の色と同じ色の石
          if (this.board[Yr][Xr] === turn) {
            //違う色の石を通ってないなら失敗
            if (dif) {
              let line = {
                direction: direction[i],
                r: r
              };
              result["line"].push(line);
            }
            break;
          }

          //盤外でも無く、空白でもなく、同じ色でもないということは
          //その桝に違う色の石があるということなのでdif=true
          dif = true;

          r++;
        }
      }

      return result;
    },

    //手番を渡す
    turnProgress() {
      this.turns_i++;
      this.turn = this.turns[this.turns_i % 2];
    },

    //手番の人が石を置く
    putStoneTurn(put_place, turn) {
      let x = put_place.x;
      let y = put_place.y;

      //裏返す
      for (let i = 0; i < put_place.line.length; i++) {
        for (let j = 0; j < put_place.line[i].r; j++) {
          //ATTENTION:jsonと配列の組み合わせなのでエラーが起きるかも
          const Yj = y + put_place.line[i].direction[0] * j;
          const Xj = x + put_place.line[i].direction[1] * j;
          this.sum[turn]++;
          this.sum[this.board[Yj][Xj]]--;
          this.board[Yj][Xj] = turn;
        }
      }
    },

    //put_ableの更新
    //TODO:this.boardに依存しているので引数にする。
    putAbleCheck: function(y, x, put_able) {
      for (let i = 0; i < put_able.length; i++) {
        if (put_able[i][0] == y && put_able[i][1] == x) {
          put_able.splice(i, 1);
          break;
        }
      }

      let direction = [
        [1, 0],
        [1, 1],
        [0, 1],
        [-1, 1],
        [-1, 0],
        [-1, -1],
        [0, -1],
        [1, -1]
      ];
      for (let i = 0; i < direction.length; i++) {
        let Xr = x + direction[i][1];
        let Yr = y + direction[i][0];

        if (
          Yr < this.N &&
          Xr < this.N &&
          Yr >= 0 &&
          Xr >= 0 &&
          this.board[Yr][Xr] === "blank"
        ) {
            //重複がないか検索してなければ追加
          let find = false;
          for (let j = 0; j < put_able.length; j++) {
            if (put_able[j][0] == Yr && put_able[j][1] == Xr) {
                //deback:alert(Yr+" "+Xr);
              find = true;
              break;
            }
          }
          if (!find) {
            put_able.push([Yr, Xr]);
          }
        }
      }
    },

    putAbleColorCheck: function(put_able, turn) {
      let put_results = [];

      for (let i = 0; i < put_able.length; i++) {
        let x = put_able[i][1];
        let y = put_able[i][0];

        let result = this.putStoneCheck(y, x, turn);

        if (typeof put_results[y] === "undefined") {
          put_results[y] = [];
        }
        put_results[y][x] = result;
      }

      return put_results;
    },

    gameEndMsg: function(white, black) {
        let msg = "draw";
      if (white > black) {
        msg = "white win!";
      }
      if (white < black) {
        msg = "black win!";
      }
      alert("game end! "+msg);
    },

    //与えられた置き結果がどこにも置けない物になっているかどうか判定する。
    checkEmptyPutResult(result){
        let result_empty = true;
          for(let i in result){
            for(let j in result[i]){
              if(result[i][j]["line"][0]){
                  result_empty = false;
                  break;
              }
            }
          }

        return result_empty;
    },

    //TODO:置いた桝に石が置けるかどうかをチェックする。
    //TODO:自動パスを作る。
    //TODO:勝利判定をする。
    onClick: function(y, x) {
      //y xが空　かつ　置いた場合に裏返せる
      if (
        this.board[y][x] == "blank" &&
        this.put_results[y][x].line.length > 0 &&
        !this.game_end
      ) {
        //更新用
        //おいた結果に応じて石をおく
        this.putStoneTurn(this.put_results[y][x], this.turn);

        //石をおける場所の更新(put_ableは配列なので参照渡しになっている)put_ableが更新される
        this.putAbleCheck(y, x, this.put_able);

        if (this.put_able.length === 0) {
          //空白がないゲーム終了
          this.gameEndMsg(this.sum["white"], this.sum["black"]);
          this.game_end = true;
        } else {
          //次ターンの手番に更新
          this.turnProgress();

          //置き場所候補に対して石を置ける場所のリストの更新
          //NOTE:ここではput_ableは更新されない。
          //参照渡しの更新はどこでされてるか分からない。
          //参照渡しの引数は綺麗な実装となりうるか？
          this.put_results = this.putAbleColorCheck(this.put_able, this.turn);

        //put_resultsに置ける場所があるかないかチェック
        //put_resultsは２重配列なので、一重で数えると、空の配列もカウントする。
        //for分で回して全部確認する。
          //置ける場所がないので自動パス
          if (this.checkEmptyPutResult(this.put_results)) {
            alert("path " + this.turn + ". There are not any places to put.");
            this.turnProgress();

            this.put_results = this.putAbleColorCheck(this.put_able, this.turn);

            //手番が変わっても置けない場合、ゲーム終了
            if (this.checkEmptyPutResult(this.put_results)) {
              this.gameEndMsg(this.sum["white"], this.sum["black"]);
              this.game_end = true;
            }
          }
        }
      }
    }
  },

  components: {
    oceloBox
  }
};
</script>