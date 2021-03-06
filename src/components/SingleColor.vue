<template>
  <el-row class="single-color">
    <el-col>
      <el-card shadow="never">
        <div class="box-card">
          <div class="color" v-bind:style="{ 'background-color': `rgb(${this.rgb.join(',')})` }"></div>

          <div class="controls">
            <el-input @change="hexColorChanged" @blur="handleChange" :value="this.hex">
              <template slot="prepend">#</template>
            </el-input>
            <el-input-number @change="handleChangeH" :min="0" :max="360" :controls="false" :value="this.h">
              <template slot="prepend">H</template>
            </el-input-number>
            <el-input-number @change="handleChangeS" :min="0" :max="100" :controls="false" :value="this.s">
              <template slot="prepend">S</template>
            </el-input-number>
            <el-input-number @change="handleChangeP" :min="0" :max="255" :controls="false" :value="this.p">
              <template slot="prepend">P</template>
            </el-input-number>
            <div class="meta">
              <span>
                Max S: {{ this.maxS }}
              </span>
              <span>
                % S: {{ this.sr.toFixed(2) }}
              </span>
              <span>
                Delta S: {{ this.maxS - this.s }}
              </span>
              <span>
                Fixed S: {{ this.fixedS }}
              </span>
            </div>
          </div>
        </div>
      </el-card>
    </el-col>
  </el-row>
</template>

<script>
import _ from "lodash";
import CSpace from "color-space";
import CString from "color-string";

export default {
  name: "SingleColor",
  props: ["id", "hspColor"],
  data() {
    const hspArr = this.hspColor.split(",");
    const rgbArr = CSpace.hsp.rgb(hspArr);

    const rgbHEX = CString.to.hex(rgbArr).slice(1);

    return {
      h: parseInt(hspArr[0], 10),
      s: parseInt(hspArr[1], 10),
      p: parseInt(hspArr[2], 10),
      r: rgbArr[0],
      g: rgbArr[1],
      b: rgbArr[2],
      hex: rgbHEX
    };
  },
  computed: {
    hsp() {
      var s =
        this.fixedS < parseInt(this.s, 10) ? this.fixedS : parseInt(this.s, 10);

      return [parseInt(this.h, 10), s, parseInt(this.p, 10)];
    },
    hsrp() {
      return [
        parseInt(this.h, 10),
        parseInt(this.sr, 10) * this.maxS / 100,
        parseInt(this.p, 10)
      ];
    },
    rgb() {
      return [parseInt(this.r, 10), parseInt(this.g, 10), parseInt(this.b, 10)];
    },
    sr() {
      return this.s / this.maxS * 100;
    },
    maxS() {
      return this.getMaxSFromHSP(this.h, this.p);
    },
    fixedS() {
      if (this.s > this.maxS) {
        let rgbArr = CSpace.hsp.rgb([this.h, this.s, this.p]);
        let fixedS = this.s;

        while (
          fixedS > 0 &&
          (rgbArr[0] > 255 || rgbArr[1] > 255 || rgbArr[2] > 255)
        ) {
          fixedS = fixedS - 1;
          rgbArr = CSpace.hsp.rgb([this.h, fixedS, this.p]);
        }

        return fixedS;
      } else {
        return this.s;
      }
    }
  },
  methods: {
    getIntSR(etalonRGB, etalonHSP, maxS) {
      let h = etalonHSP[0];
      let sr = etalonHSP[1] / maxS * 100;
      let p = etalonHSP[2];

      if (sr === parseInt(sr, 10)) {
        return sr;
      }

      let roundSR = Math.round(sr);

      if (_.isEqual(CSpace.hsp.rgb([h, roundSR * maxS / 100, p]), etalonRGB)) {
        return roundSR;
      }

      let floorSR = Math.floor(sr);

      if (_.isEqual(CSpace.hsp.rgb([h, floorSR * maxS / 100, p]), etalonRGB)) {
        return floorSR;
      }
    },
    handleChangeH(next) {
      this.h = next;
      this.handleChange();
    },
    handleChangeS(next) {
      this.s = next;
      this.handleChange();
    },
    handleChangeSR(next) {
      this.sr = next;
      this.handleChange();
    },
    handleChangeP(next) {
      this.p = next;
      this.handleChange();
    },
    handleChange() {
      let rgbArr = CSpace.hsp.rgb(this.hsp);

      this.r = rgbArr[0];
      this.g = rgbArr[1];
      this.b = rgbArr[2];

      const rgbHEX = CString.to.hex(rgbArr);

      if (rgbHEX) {
        this.hex = rgbHEX.slice(1);

        this.$emit("update", {
          id: this.id,
          value: [this.h, this.s, this.p].join(",")
        });
      } else {
        this.hex = "Wrong HEX";
      }
    },
    getMaxSFromHSP(h, p) {
      let maxS = 0;

      let rgbArr = CSpace.hsp.rgb([h, maxS, p]);

      while (rgbArr[0] <= 255 && rgbArr[1] <= 255 && rgbArr[2] <= 255) {
        if (maxS === 100) {
          maxS = maxS + 1;
          break;
        }
        maxS = maxS + 1;
        rgbArr = CSpace.hsp.rgb([h, maxS, p]);
      }

      maxS = maxS - 1;

      return maxS;
    },
    hexColorChanged(next) {
      this.hex = next;
      const rgbArr = CString.get.rgb(`#${this.hex}`);

      if (rgbArr) {
        this.r = rgbArr[0];
        this.g = rgbArr[1];
        this.b = rgbArr[2];

        const hspArr = CSpace.rgb.hsp(this.rgb);

        this.h = parseInt(hspArr[0], 10);
        this.s = parseInt(hspArr[1], 10);
        this.p = parseInt(hspArr[2], 10);

        this.fixedS = this.s;
      }
    },
    makeCleanColor() {
      this.s = this.maxS;
      this.handleChange();
    }
  },
  mounted() {
    this.handleChange();
  }
};
</script>

<style lang="less">
.single-color {
  .el-input__inner,
  .el-input-group__prepend {
    /*color: red;*/
    border: none;
    padding-right: 0;
  }

  .el-input-group__prepend {
    background-color: inherit;
  }

  .el-card__body {
    padding: 0;
  }

  .el-card {
    box-shadow: none;
    border: none;
    border-radius: 0;
  }

  .el-input,
  .el-input-number,
  .el-input-group {
    width: 100px;
    margin-left: 12px;
  }

  .box-card {
    display: flex;
    align-items: stretch;
  }
}

.color {
  width: 100px;
  min-height: 50px;
  display: inline-block;
}

.controls {
  display: inline-block;
}

.color__delta {
  cursor: pointer;
}

.meta {
  margin-left: 32px;
  font-size: 12px;
}
</style>
