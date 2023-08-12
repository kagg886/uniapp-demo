<template>
	<div class="container">
		<div class="header">课程表(第{{currentWeek}}周)</div>
		<div class="table">
			<div class="row">
				<div class="cell time">课节</div>
				<div class="cell day">星期一</div>
				<div class="cell day">星期二</div>
				<div class="cell day">星期三</div>
				<div class="cell day">星期四</div>
				<div class="cell day">星期五</div>
				<div class="cell day">星期六</div>
				<div class="cell day">星期日</div>
			</div>
			<!-- 添加课程表数据 -->
			<div class="row" v-for="section in 8">
				<div class="cell time">{{ section }}</div>
				<div v-for="day in 7" class="cell day">
					<div v-for="(v,k) in findCourseBySectionAndWeek(section,day)">
						<div v-if="k === 'name' || k == 'location'">{{v}}</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		methods: {
			findCourseBySectionAndWeek(section, day) {
				let classResult = this.courseData.filter(s => s.section === section && s.weekday == day && s.weeks
					.includes(this.currentWeek))[0];
				// {
				// 	"classId": "210305612",
				// 	"name": "人工智能实例与应用",
				// 	"teacher": "程磊,吴嘉轩,刘昶,史天予",
				// 	"category": "专理选",
				// 	"method": "考查",
				// 	"location": "XX-428",
				// 	"section": 1,
				// 	"sectionCount": 2,
				// 	"weekday": 2,
				// 	"weeks": [9, 10, 11, 12, 13, 14, 15, 16]
				// }
				// console.log(classResult.name)
				// let name = classResult.name
				// let location = classResult.location
				// let teacher = classResult.teacher

				return classResult === undefined ? [] : classResult
			}
		},

		async created() {

			const token = uni.getStorageSync('userInfo')

			const resp = await uni.request({
				url: 'https://openapi.hackerxiao.online/api/v1/edu/courses',
				header: {
					Authorization: 'Bearer ' + token,
					'Content-Type': 'application/json',
				},
			})

			this.courseData = resp.data.courses;

			function dateChange(num = 1, date = false) {
				if (!date) {
					date = new Date(); //没有传入值时,默认是当前日期
					date = date.getFullYear() + '-' + (date.getMonth() + 1) + '-' + date.getDate();
				}
				date += " 00:00:00"; //设置为当天凌晨12点
				date = Date.parse(new Date(date)) / 1000; //转换为时间戳
				date += (86400) * num; //修改后的时间戳
				var newDate = new Date(parseInt(date) * 1000); //转换为时间
				return new Date(newDate.getFullYear(), (newDate.getMonth() + 1), newDate.getDate());
			}

			let start = new Date(Object.values(resp.data.starttime))
			while ((start = dateChange(7, start)).getTime() < Date.now()) {
				this.currentWeek++
			}
			console.log('course loaded!')
		},

		data() {
			return {
				courseData: [],
				currentWeek: 1,
			};
		},
	}
</script>

<style>
	.container {
		padding: 20rpx;
		background-color: #f7f7f7;
		border-radius: 10rpx;
		box-shadow: 0px 2px 6px rgba(0, 0, 0, 0.1);
	}

	.header {
		font-size: 36rpx;
		font-weight: bold;
		text-align: center;
		margin-bottom: 20rpx;
		color: #333;
		background-color: #eee;
		padding: 10rpx;
		border-radius: 5rpx;
	}

	.table {
		display: flex;
		flex-direction: column;
		margin-top: 20rpx;
	}

	.row {
		display: flex;
	}

	.cell {
		flex: 1;
		text-align: center;
		border: none;
		padding: 10rpx;
		background-color: #fff;
		border-radius: 5rpx;
		box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
		margin: 5rpx;
	}

	.time {
		font-weight: bold;
		background-color: #eee;
	}

	.day {
		background-color: #eee;
	}
</style>