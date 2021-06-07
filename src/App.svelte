<script>
	import Chart from 'svelte-frappe-charts';
	import * as data from "../public/1000Beers.json";

	function formatDateUk(date){
		var ukDate = date.split(' ');
		var ukTimeParts = ukDate[1].split(":");
		var hour = parseInt(ukTimeParts[0], 10), minute = parseInt(ukTimeParts[1], 10);
		var ukDateParts = ukDate[0].split("/");
    	var day = parseInt(ukDateParts[0], 10), month = parseInt(ukDateParts[1], 10), year = parseInt(ukDateParts[2], 10);
    	return new Date(year, month - 1, day, hour, minute);
	}

	function count(list, item){
		if(item == "" || item == "Untappd at Home"){
			return;
		}

		let found = list.find(v => v.name == item)
		if(found){
			found.count++;
		} else {
			list.push({name:item, count:1});
		}
	}

	function groupBeers(list, item){
		if(item.includes("pale") || item.includes("Pale") || item.includes("IPA")){
			let found = list.find(b => b.name == "Pale");
			if(found){
				found.count++;
			} else {
				list.push({name:"Pale", count:1});
			}
		} else {
			let found = list.find(v => v.name == item)
			if(found){
				found.count++;
			} else {
				list.push({name:item, count:1});
			}
		}
	}

	function findAverage(list){
		var total = 0;
		for(var i = 0; i < list.length; i++) {
    		total += list[i];
		}

		return Math.round(total / list.length * 100) / 100;
	}

	let firstCheckin = data.default[0].created_at;	
	let lastCheckin = data.default[data.default.length-1].created_at;
	let numberOfDaysToComplete = Math.round((formatDateUk(lastCheckin)-formatDateUk(firstCheckin))/(1000*60*60*24));
	let averageBeerPerDay = Math.round(data.default.length/numberOfDaysToComplete * 100) / 100;

	let months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
	let monthValues = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
	let days = ['Sun', 'Mon', 'Tues', 'Weds', 'Thurs', 'Fri', 'Sat'];
	let dayValues = [0, 0, 0, 0, 0, 0, 0];
	let years = ["2014", "2015", "2016", "2017", "2018", "2019", "2020", "2021"];
	let yearValues = [0, 0, 0, 0, 0, 0, 0, 0];
	let hours = ["00", "01", "02", "03", "04", "05", "06", "07", "08", "09", "10", "11", "12", "13", "14", "15", "16", "17", "18", "19", "20", "21", "22", "23"];
	let hourValues = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0];

	let venues = [];

	let beerTypes = [];
	let beweries = [];
	let beweryCountries = [];

	let ratings = [];
	let averageRating = [];
	let globalRatings = [];
	let averageGlobalRating = [];

	for (const checkIn of data.default) {
		let checkInDate = formatDateUk(checkIn.created_at);
		monthValues[checkInDate.getMonth()]++;
		dayValues[checkInDate.getDay()]++;
		yearValues[checkInDate.getFullYear()-2014]++;
		hourValues[checkInDate.getHours()]++;

		count(venues, checkIn.venue_name);
		venues.sort(function(a, b) { return b.count - a.count; });

		groupBeers(beerTypes, checkIn.beer_type);
		beerTypes.sort(function(a, b) { return b.count - a.count; });

		count(beweries, checkIn.brewery_name);
		beweries.sort(function(a, b) { return b.count - a.count; });
		count(beweryCountries, checkIn.brewery_country);
		beweryCountries.sort(function(a, b) { return b.count - a.count; });

		ratings.push(checkIn.rating_score);
		globalRatings.push(checkIn.global_rating_score);
	}

	averageRating = findAverage(ratings);
	averageGlobalRating = findAverage(globalRatings);

	let monthData = {
    	labels: months,
		datasets: [
		{
			values: monthValues, chartType: 'bar'
		}]
  	};
	let dayData = {
    	labels: days,
		datasets: [
		{
			values: dayValues, chartType: 'bar'
		}]
  	};
	let yearData = {
    	labels: years,
		datasets: [
		{
			values: yearValues, chartType: 'bar'
		}]
  	};
	let hourData = {
    	labels: hours,
		datasets: [
		{
			values: hourValues, chartType: 'bar'
		}]
  	};
	let chartColour = ['#FFC000'];
</script>

<main>
	<header>
		<p>1000 Beers</p>
	</header>

	<section>
		<div>
			<h1>First Checkin: </h1>
			<p>{firstCheckin}</p>
		</div>
		<div>
			<h1>Last Checkin: </h1>
			<p>{lastCheckin}</p>
		</div>
		<div>
			<h1>Number of Days to Complete: </h1>
			<p>{numberOfDaysToComplete}</p>
		</div>
		<div>
			<h1>Average Beer Per Day: </h1>
			<p>{averageBeerPerDay}</p>
		</div>
		<div>
			<h1>Average Rating: </h1><p>{averageRating}</p>
		</div>
		<div>
			<h1>Average Global Rating: </h1><p>{averageGlobalRating}</p>
		</div>
	</section>

	<div class="charts">
		<Chart data={monthData} title="Beers by Month" colors={chartColour}/>
		<Chart data={dayData} title="Beers by Day" colors={chartColour}/>
		<Chart data={yearData} title="Beers by Year" colors={chartColour}/>
		<Chart data={hourData} title="Beers by Hour" colors={chartColour}/>
	</div>

	<section>
		<div>
			<h1>Most Visited Venues: </h1>{#each venues.slice(0, 10) as venue, i} <p>{venue.name} - {venue.count}</p> {/each}
		</div>
		<div>
			<h1>Beers per Brewery: </h1>{#each beweries.slice(0, 10) as bewery, i} <p>{bewery.name} - {bewery.count}</p> {/each}
		</div>
		<div>
			<h1>Beers per Country: </h1>{#each beweryCountries.slice(0, 10) as beweryCountries, i} <p>{beweryCountries.name} - {beweryCountries.count}</p> {/each}
		</div>
		<div>
			<h1>Beers per Type: </h1>{#each beerTypes.slice(0, 10) as beerType, i} <p>{beerType.name} - {beerType.count}</p> {/each}
		</div>
	</section>
</main>

<style>
	header {
		margin: 0 0 1rem 0;
		padding: 7rem 2rem 0.75rem 1rem;

		background: url("../images/banner.jpeg");
		background-repeat: no-repeat;
		background-size: cover;
		background-position: bottom;
	}

	header > p {
		margin: 0;
		font-size: 2rem;
		vertical-align: bottom;

		color: white;
	}

	section {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-around;
		gap: 1rem;
	}

	section > div > h1 {
		background: #FFC000;
		padding: 0.1rem 1rem;
		margin: 0;

		font-size: 1.2rem;
	}

	section > div > p {
		margin-top: 0.2rem;
		text-align: center;
	} 

	@media only screen and (min-width: 700px) {
		header, section, .charts  {
			width: 75%;
			margin: 0 auto 1rem auto;
			max-width: 800px;
		}
	}
</style>