<template>
    <div class="container">
        <h1>회원 정보 조회</h1>
        <div class="row">
            <table class="table">
                <tr>
                    <th class="text-right table-primary">No.</th>
                    <td class="text-center">{{ userInfo.user_no }}</td>
                </tr>
                <tr >
                    <th class="text-right table-primary">ID</th>
                    <td class="text-center">{{ userInfo.user_id }}</td>
                </tr>
                <tr>
                    <th class="text-right table-primary">비밀번호</th>
                    <td class="text-center">{{ userInfo.user_pwd }}</td>
                </tr>
                <tr>
                    <th class="text-right table-primary">이름</th>
                    <td class="text-center">{{ userInfo.user_name }}</td>
                </tr>
                <tr>
                    <th class="text-right table-primary">성별</th>
                    <td class="text-center">{{ userGender }}</td>
                </tr>
                <tr>
                    <th class="text-right table-primary">나이</th>
                    <td class="text-center">{{ userInfo.user_age }}</td>
                </tr>
                <tr>
                    <th class="text-right table-primary">가입일자</th>
                    <td class="text-center">{{ joinDate }}</td>
                </tr>
            </table>
        </div>
    </div>
        <div class="row">
            <button class="btn btn-info col-4" @click="goToUpdate(userInfo.user_id)">수정</button>
            <router-link to="/" class="btn btn-success col-4">목록</router-link>
            <button class="btn btn-warning col-4" @click="deleteInfo(userInfo.user_id)">삭제</button>
        </div>
</template>

<script>
import axios from 'axios'

export default {
    data(){
        return {
            userInfo : {}
        }
    },
    computed : {
        //성별
        userGender(){
            let result = null;
            if(this.userInfo.user_gender == "M"){
                result = "남성";
            }else if(this.userInfo.user_gender == "F"){
                result = "여성";
            }
            return result;
        },
        //다른 방식으로 성별 구하는 방법.
        // userGender(){
        //     let map = {
        //         "M" : "남",
        //         "F" : "여"
        //     };
        //     return map[this.userInfo.user_gender];
        // },

        //가입날짜 : 년 월 일
        joinDate(){
            let result = null;
            if(this.userInfo.join_data != null){
                let date = new Date(this.userInfo.join_data);
                let year = date.getFullYear();
                let month = ('0' + (date.getMonth() +1)).slice(-2);
                let day = ('0' + date.getDay()).slice(-2);

                result  = `${year}년${month}월${day}일`;
                30
            }
            return result;
        }
    },
    created(){
        let searchNo = this.$route.query.userId;
        this.getUserInfo(searchNo);
    },
    methods : {
        async getUserInfo(userId){
        //http://localhost:3000/users/asmin
            let result = await axios.get('/api/users' + userId)
                                    .catch(err => console.log(err));
            let info = result.data;
            this.userInfo = info;
        },
        goToUpdate(userId){
            //수정폼 컨포너트 호출
            console.log(userId);
            this.$route.push({name : 'userUpdate', query : {'userId': userId}});
        },
        deleteInfo(userId){
            //서버에 해당 데이터를 삭제
            console.log(userId);
        }
    }
}
</script>