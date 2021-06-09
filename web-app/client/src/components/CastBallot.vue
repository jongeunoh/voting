<template>
  <div class="posts">
    <h1>Cast Ballot</h1>
    <input type="radio" id="one" value="후보자1" v-model="picked">
    <label for="one">후보자1</label>
    <br>
    <input type="radio" id="two" value="후보자2" v-model="picked">
    <label for="two">후보자2</label>
    <br>
    <input type="radio" id="three" value="후보자3" v-model="picked">
    <label for="three">후보자3</label>
    <br>
    <input type="radio" id="four" value="후보자4" v-model="picked">
    <label for="four">후보자4</label>
    <br>
    <input type="radio" id="five" value="후보자5" v-model="picked">
    <label for="five">후보자5</label>
    <br>
    <br>
    <span v-if="picked">
      Picked:
      <b>{{ picked }}</b>
    </span>
    <br>
    <br>
    <!--span><b>{{ response }}</b></span><br /-->
    <form v-on:submit="castBallot">
      <input type="submit" value="투표하기">
      <br>
    </form>

    <br>
    <span v-if="response">
      <b>{{ response }}</b>
    </span>
    <br>
    <vue-instant-loading-spinner id="loader" ref="Spinner"></vue-instant-loading-spinner>
  </div>
</template>

<script>
import PostsService from "@/services/apiService";
import VueInstantLoadingSpinner from "vue-instant-loading-spinner/src/components/VueInstantLoadingSpinner.vue";

export default {
  name: "response",
  data() {
    return {
      input: {},
      picked: null,
      response: null,
      token: null
    };
  },
  created(){
    this.token = this.$route.params.tokenId
    console.log("this.$route.params.tokenId ${this.$route.params.tokenId}")
  },
  components: {
    VueInstantLoadingSpinner
  },
  methods: {
    clickHandler(token){
      console.log("Oh, that's nice. It's gotten ${token} got! :) " );
      this.token = token;
    },

    async castBallot() {
      await this.runSpinner();

      // const electionRes = await PostsService.queryWithQueryString('election');

      // let electionId = electionRes.data[0].Key;

      console.log("picked: ");
      console.log(this.picked);
      // console.log("voterId: ");
      // console.log(this.input.voterId);
      // this.response = null;

 

      //error checking for making sure to vote for a valid party
      if (this.picked === null ) {
        console.log('this.picked === null')

        let response = "후보자를 투표해주세요";
        this.response = response;
        await this.hideSpinner();
      
      // } else if (this.input.voterId === undefined) {
      //   console.log('this.voterId === undefined')

      //   let response = "투표를 진행하기 위해 투표자 코드를 입력하세요";
      //   this.response = response;
      //   await this.hideSpinner();

      } 
      else {
          console.log("token in castBallot: ");
          console.log(this.token);
        const apiResponse = await PostsService.castBallot(
          null,
          this.token,
          this.picked
        );

        console.log('apiResponse: &&&&&&&&&&&&&&&&&&&&&&&');
        console.log(apiResponse);

        if (apiResponse.data.error) {
          this.response= apiResponse.data.error;
          await this.hideSpinner();
        } else if (apiResponse.data.message) {
          this.response= apiResponse.data.message
          console.log(this.response);
          this.$router.push({name: "GetCurrentStanding",params: {tokenId: this.token}});
          await this.hideSpinner();
        } 
        else {
          let response = `${this.picked} 가 정상적으로 투표되었습니다. 
            투표자 코드 ${apiResponse.data.voterId} 님 투표 감사합니다.`;

          this.response = response;

          console.log("cast ballot");
          console.log(this.input);
          await this.hideSpinner();
        }
      }
    },
    async runSpinner() {
      this.$refs.Spinner.show();
    },
    async hideSpinner() {
      this.$refs.Spinner.hide();
    }
  }
};
</script>
