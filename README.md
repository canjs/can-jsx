# can-jsx

[![Build Status](https://travis-ci.org/canjs/can-jsx.svg?branch=master)](https://travis-ci.org/canjs/can-jsx)

CanJS + JSX is designed to combine popular features of different frameworks:

* Custom Elements
* JavaScript templating
* Intuitive event and data bindings
* Object-oriented ViewModels
* Views that update automatically when the ViewModel changes

Here is an example of what this would look like:

```js
import Component from "can-component";
import { h, render } from "can-jsx";

Component.extend({
	tag: "weather-report",
	
	ViewModel: {
	  location: {
			type: "string",
			set: function(){
				this.place = null;
			}
		},
		get placesPromise(){
			if(this.location && this.location.length > 2) {
				return fetch(
					yqlURL +
					can.param({
						q: 'select * from geo.places where text="'+this.location+'"',
						format: "json"
					})
				).then(function(response){
					return response.json();
				}).then(function(data){
					console.log(data);
					if(Array.isArray(data.query.results.place)) {
						return data.query.results.place;
					} else {
						return [data.query.results.place];
					}
				});
			}
		},
		places: {
			get: function(lastSet, resolve) {
				if(this.placesPromise) {
					this.placesPromise.then(resolve)
				}
			}
		},
		get showPlacePicker(){
			return !this.place && this.places && this.places.length > 1;
		},
		place: {
			type: "any",
			get: function(lastSet){
				if(lastSet) {
					return lastSet;
				} else {
					if(this.places && this.places.length === 1) {
						return this.places[0];
					}
				}
			}
		},
		pickPlace: function(place){
			this.place = place;
		},
		get forecastPromise(){
			if( this.place ) {
				return fetch(
						yqlURL+
						can.param({
							q: 'select * from weather.forecast where woeid='+this.place.woeid,
							format: "json"
						})
				).then(function(response){
						return response.json();
				}).then(function(data){
					console.log("forecast data", data);
					const forecast = data.query.results.channel.item.forecast;

					return forecast;
				});
			}
		},
		toClassName: function(text){
			return text.toLowerCase().replace(/ /g, "-");
		}
	},
	
	view: render(() => {
		const toClassName = text => text.toLowerCase().replace(/ /g, "-");
		
		return 
			<div class="weather-widget">
					<div class="location-entry">
							<label for="location">Enter Your location:</label>
							<input id="location" value:to="vm.location" type="text"/>
					</div>

					{ placesPromise.isPending &&
							<p class="loading-message">
									Loading places…
							</p>
					}

					{ data.showPlacePicker &&
							<div class="location-options">
									<label>Pick your place:</label>
									<ul>
											{ places.map(place =>
													<li on:click="data.pickPlace(place)">{place.name}</li>
											)}
									</ul>
							</div>
					}

					{ data.place &&
							<div class="forecast">
									<h1>10 day {data.place.name} Weather Forecast</h1>
									<ul>
											{data.place.forecasts.map(forecast =>
													<li>
															<span class='date'>{forecast.date}</span>
															<span class={`description ${toClassName(forecast.text)}`}>{forecast.text}</span>
															<span class='high-temp'>{forecast.high}<sup>&deg;</sup></span>
															<span class='low-temp'>{forecast.low}<sup>&deg;</sup></span>
													</li>
											)}
									</ul>
							</div>
					}
			</div>
	})
});
```

For Technical details on how this would work, see [canjs/canjs Proposal: Support JSX](https://github.com/canjs/canjs/issues/3569).
