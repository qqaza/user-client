<!-- 등록과 수정을 하나로 합하는것-->
<template>
    <div class="container">
     <h3 class="text-center">{{ title }}</h3>
     <div class="row">
       <table class="table">
         <tr>
           <th class="text-right table-primary">No.</th>
           <td class="text-center">
             <input class="form-control" type="number" v-model="userInfo.user_no" readonly>
           </td>
         </tr>
         <tr>
           <th class="text-right table-primary">ID</th>
           <td class="text-center">
             <input class="form-control" type="text" v-model="userInfo.user_id" v-bind:readonly="isUpdated">
           </td>
         </tr>
         <tr>
           <th class="text-right table-primary">비밀번호</th>
           <td class="text-center">
             <input class="form-control" type="password" v-model="userInfo.user_pwd">
           </td>
         </tr>
         <tr>
           <th class="text-right table-primary">이름</th>
           <td class="text-center">
             <input class="form-control" type="text" v-model="userInfo.user_name">
           </td>
         </tr>
         <tr>
           <!-- 남녀 -->
           <th class="text-right table-primary">성별</th>
           <td class="text-center">
             <!-- vue에서 id, name 속성 사용 안함 -->
             <input type="radio" value="M" v-model="userInfo.user_gender"> 남
             <input type="radio" value="F" v-model="userInfo.user_gender"> 여
             <!-- <input true-value="예" false-value="아니오" v-model="chkVal"> -->
           </td>
         </tr>
         <tr>
           <th class="text-right table-primary">나이</th>
           <td class="text-center">
             <input class="form-control" type="number" min="0" v-model="userInfo.user_age">
           </td>
         </tr>
         <tr>
           <!-- 년월일 -->
           <th class="text-right table-primary">가입일자</th>
           <td class="text-center">
             <input class="form-control" type="date" v-model="userInfo.join_date"> <!-- yyyy-MM-dd -->
           </td>
         </tr>
       </table>
     </div>
     <div class="row">
        <!--첫번째-->
      <!-- <button class="btn btn-info" @click="saveInfo(searchNo)">저장</button>-->
       <!--두번째-->
       <button class="btn btn-info" @click="isUpdated ? updateInfo() : insertInfo()">저장</button>
     </div>
   </div>
 </template>

 <script>
 import axios from 'axios'
export default{
    data(){
        return{
            userInfo: {
                user_no : null,
                user_id : "",
                user_pwd : "",
                user_name : "",
                user_age : "",
                user_gender : null,
                join_date : null
            },
            searchNo : null,  //등록 OR 수정 (넘어오는 값이 있으면 수정, 넘어오는 값이 없으면 등록)
            isUpdated : false
        }
    },
    computed : {
        title(){
            return this.isUpdated ? '회원 정보 수정' : '회원 정보 등록';
        }
    },
    created(){
        this.searchNo = this.$route.query.userId;
        if(this.searchNo != null && this.searchNo != undefined){
            this.getUserInfo();
        }else{
            this.userInfo.join_date_date = this.getDate('');
        }
    },
    methods : {
      async  getUserInfo(){
        let result = await axios.get(`/api/users/${this.searchNo}`)
                                .catch(err => console.log(err));
            let info = result.data;

        let newDate = this.getDate(info.join_date);
        info.join_date = newDate;

            this.userInfo = info; 
            this.isUpdated = true;
        },
        getDate(value){
        if(value == null) return null;
        
            let date = value == '' ? new Date() : new Date(value);
            let year = date.getFullYear();
            let month = ('0' + (date.getMonth()+1)).slice(-2);
            let day = ('0' + date.getDate()).slice(-2);

            return `${year}-${month}-${day}`;
        },
        saveInfo(user_id){ //첫번째
            //1)보낼 데이터 산출
            let info = this.getSendInfo(user_id);

            //2)ajax
            axios(info)
            .then(result =>{
                let count = result.data.affectedRows;
                if(count == 0){
                    alert('저장되지 않았습니다. 내용을 확인해주세요.');
                }else{
                    alert('저장되었습니다.');

                    if(result.data.insertId > 0 ){ // 등록일 경우 추가 작업
                        this.userInfo.user_no = result.data.insertId;
                    }
                }
            })
            .catch(err => console.log(err));
        },
        getSendInfo(id){
            //method, rul, data 결정해야함.
            let method = '';
            let url = '';
            let data = null;

            if(id == null || id == undefined){ // 등록
                method = 'post';
                url = '/api/users';
                data = {
                    "param" : {
                        user_id : this.userInfo.user_id,
                        user_pwd : this.userInfo.user_pwd,
                        user_name : this.userInfo.user_name,
                        user_age : this.userInfo.user_age,
                        user_gender : this.userInfo.user_gender,
                        join_date : this.userInfo.join_date
                    }
                };
            }else{ // 수정
                method = 'put';
                url = `/api/users/${id}`;
                data = {
                    "param" : {
                        user_pwd : this.userInfo.user_pwd,
                        user_name : this.userInfo.user_name,
                        user_age : this.userInfo.user_age,
                        user_gender : this.userInfo.user_gender,
                        join_date : this.userInfo.join_date
                    }
                };
            }
            return{
                method,
                url,
                data
            }
        },
        //두번째 방법
        insertInfo(){
            if(!this.validation()) return;

            let data = {
                "param" :  {
                        user_id : this.userInfo.user_id,
                        user_pwd : this.userInfo.user_pwd,
                        user_name : this.userInfo.user_name,
                        user_gender : this.userInfo.user_gender,
                        user_age : this.userInfo.user_age,
                        join_date : this.userInfo.join_date
                    }
            }

            axios
            .post('/api/users', data)
            .then(result => {
                let user_no = result.data.insertId;
                if(user_no == 0){
                    alert(`등록되지 않았습니다.\n내용을 확인해주세요`)
                }else{
                    alert(`정상적으로 등록되었습니다.`);
                    this.userInfo.user_no = user_no;
                }
            })
            .catch(err => console.log(err));        

        },
        updateInfo(){
            if(!this.validation()) return;

            let data = {
                "param" : {
                        user_pwd : this.userInfo.user_pwd,
                        user_name : this.userInfo.user_name,
                        user_gender : this.userInfo.user_gender,
                        user_age : this.userInfo.user_age,
                        join_date : this.userInfo.join_date
                    }
            };

            axios
            .put(`/api/users/${this.userInfo.user_id}`, data)
            .then(result => {
                let count = result.data.changedRows;
                if(count == 0){
                    alert(`수정되지 않았습니다.\n내용를 확인해주세요`)
                }else{
                    alert(`정상적으로 수정되었습니다.`);
                }                   
            })
            .catch(err => console.log(err));        

        },
        validation(){
            if(this.userInfo.user_id == "" && !this.isUpdated){
                alert('아이디가 입력되지 않았습니다.');
                return false;
            } 

            if(this.userInfo.user_pwd == ""){
                alert('비밀번호가 입력되지 않았습니다.');
                return false;
            }

            if(this.userInfo.user_name == ""){
                alert('이름이 입력되지 않았습니다.');
                return false;
            }

            return true;
        }
    }
}
</script>