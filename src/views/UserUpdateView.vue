<!-- UserInsertView.vue -->
<template>
    <div class="container">
     <h1>회원 정보 수정</h1>
     <div class="row">
       <table class="table">
         <tr>
           <th class="text-right table-primary">No.</th>
           <td class="text-center">
             <input class="form-control" type="number"  v-model="userInfo.user_no" readonly>
           </td>
         </tr>
         <tr>
           <th class="text-right table-primary">ID</th>
           <td class="text-center">
             <input class="form-control" type="text" v-model="userInfo.user_id" readonly>
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
           <th class="text-right table-primary">성별</th>
           <td class="text-center">
             <input type="radio" value="M" v-model="userInfo.user_gender"> 남성
             <input type="radio" value="F" v-model="userInfo.user_gender"> 여성
           </td>
         </tr>
         <tr>
           <th class="text-right table-primary">나이</th>
           <td class="text-center">
             <input class="form-control" type="number" min="0" v-model="userInfo.user_age">
           </td>
         </tr>
         <tr>
           <th class="text-right table-primary">가입일자</th>
           <td class="text-center">
             <input class="form-control" type="date" v-model="userInfo.join_date"> <!-- yyyy-MM-dd -->
           </td>
         </tr>
       </table>
     </div>
     <div class="row">
       <button class="btn btn-info" @click="updateInfo()">저장</button>
     </div>
   </div>
 </template>
 
 <script>
 import axios from 'axios'
 
 export default {
   data() {
     return {
       userInfo: {
         user_no : null,
         user_id : "",
         user_pwd : "",
         user_name : "",
         user_age : "",
         user_gender : null,
         join_date : null
       }
     }
   },
   created() {
     let searchNo = this.$route.query.userId;
     this.getUserInfo(searchNo);
   },
   methods : {
     async getUserInfo(userId) {
       let result = await axios.get('/api/users/' + userId)
                               .catch(err => console.log(err));
       let info = result.data;
       //날짜 포멧변경해서 yyyy-MM-dd 로 보이게 하기.
       let newDate = this.dateFormat(info.join_date);
       info.join_date = newDate;

       this.userInfo = info; 
     },
     dateFormat(value){
        let result = null;
        if(value != null){
            let date = new Date(value);
            let year = date.getFullYear();
            let month = ('0' + (date.getMonth()+1)).slice(-2);
            let day = ('0' + date.getDate()).slice(-2);
            result = `${year}-${month}-${day}`;
        }
        return result;
     },
     updateInfo() {
       // 1) 유효성 체크
       if(!this.validation()) return;
 
       // 2) ajax
       // 2-1) 실제 보낼 데이터 선별
       let data = this.getSendData();
 
       // 2-2) axios 이용해 ajax
       axios
       .put('/api/users/'+ this.userInfo.user_id, data)
       .then(result => {
         console.log(result);
         // 3) 결과 처리
         let user_no = result.data.changedRows;
         if(user_no == 0){
           alert(`수정되지 않았습니다.\n메세지를 확인해주세요\n${result.data.message}`);
         }
         else {
           alert(`정상적으로 수정되었습니다.`);
           this.$router.push({ path: '/userInfo', query: {'userId' : this.userInfo.user_id}});
         }
       })
       .catch(err => console.log(err));
     },
     validation() {
       if(this.userInfo.user_pwd == '') {
         alert('비밀번호가 입력되지 않았습니다.');
         return false;
       }
       if(this.userInfo.user_name == '') {
         alert('이름이 입력되지 않았습니다.');
         return false;
       }
       return true;
     },
     getSendData() {
       let obj = this.userInfo;
       let delData = ["user_no", "user_id"];
       let newObj = {};
 
       let isTargeted = null;
       for(let field in obj){
           isTargeted = false;
           for(let target of delData){
               if(field == target){
                 isTargeted = true;
                 break;
               }
           }
           if(!isTargeted){
             newObj[field] = obj[field];
           }
       }
       let newData = { 
         "param" : newObj
       }
       return newData;
     }
   }
 }
 </script>