Assignment - README.TXT file explaining the project.

I was asked to create a simple application using a flickr API which would display a search feature with text input with a button to click , once button is clicked
the field should show relevant pictures to the search word and also once a picture is clicked there is a redirect with the picture in full screen with details of image.

Resources

JQuery Mobile theme - i used the theme because I felt JQuery Mobile is suitable for a Mobile Application and it also allows you to Develop and Modify code without restrictions.
Search API example : https://github.com/yannickcr/flickr-search - i felt this guys demonstration was incredible and the code worked so i felt it was the best way 
of designing this assesment and also it followed all the requirements of the assesment.

How I created the application

1. Created the theme and downloaded it.

2. Extracted the file to the appropriate area.

3. added the following HTML to the website:

	<h1>Welcome to my Example App</h1>
			<p>This is an example of being able to search for files on flickr</p>
			<form class="search-form" method="get" action="/">
			<input id="search-form-q" type="search" placeholder="Type a keyword to search on Flickr" />
			<button type="submit">Search</button>
		</form>
		<ul class="list"></ul>
		<div class="loader"></div>
		<div class="cover"></div>
		

4. created the appropriate Javascript and named the javascript main.js:
(function() {

	// Create the card template (convert a string in DOM Node and store it in tpl)
	var tpl = document.createElement('ul');
	tpl.innerHTML =
		'<li class="card flipped">' +
		'	<a class="front">' +
		'		<img class="photo" width="150" height="150" />' +
		'	</a>' +
		'</li>'
	;
	tpl = tpl.firstChild;

	// Init variables
	var nextPage, hasScrollRequest = false;

	// Inialize the Flickr API
	var flickr = new Flickr({
		api_key: '46dedd06fca73ef6da2dafd9c5e1bc3c'
	});

	/*
	 * Define functions
	 */

	// "Empty" the results list
	var emptyResults = function() {
		Array.prototype.slice.call(document.querySelectorAll('.card')).forEach(function(card) {
			card.classList.add('flipped');
		});
	};

	// Replace the content of the list
	var replaceResults = function(err, data) {
		if (err) throw new Error(err.message);

		// Remove the loader
		document.querySelector('.loader').classList.remove('active');

		hasScrollRequest = false;

		var cards = Array.prototype.slice.call(document.querySelectorAll('.card'));

		var items = document.createDocumentFragment();

		data.photos.photo.forEach(function(photo, i) {
			var item, newItem = false;
			// Get an existing item
			if (cards[i]) {
				item = cards[i];
			// create a new one using the template
			} else {
				item = tpl.cloneNode(true);
				newItem = true;
			}

			// Update the item / Fill the template
			item.querySelector('.front').setAttribute('href', 'http://www.flickr.com/photos/' + photo.owner + '/' + photo.id);
			item.querySelector('.photo').setAttribute('src', (photo.url_q || photo.url_t) + '?ts=' + (+new Date()));
			item.querySelector('.photo').setAttribute('alt', photo.title);
			item.querySelector('.photo').setAttribute('title', photo.title);

			if (newItem) items.appendChild(item);
		});

		// Append the new cards to the list
		document.querySelector('.list').appendChild(items);

		// Remove the un-needed cards from the list
		if (cards.length - data.photos.photo.length <= 0) return;
		Array.prototype.slice.call(document.querySelectorAll('.card:nth-child(1n+' + data.photos.photo.length + ')')).forEach(function(card) {
			card.parentNode.removeChild(card);
		});
	};

	// Append the new results at the end of the list
	var appendResults = function(err, data) {
		if (err) throw new Error(err.message);

		hasScrollRequest = false;

		var items = document.createDocumentFragment();

		data.photos.photo.forEach(function(photo, i) {
			var item = tpl.cloneNode(true);

			// Update the item / Fill the template
			item.querySelector('.front').setAttribute('href', 'http://www.flickr.com/photos/' + photo.owner + '/' + photo.id);
			item.querySelector('.photo').setAttribute('src', (photo.url_q || photo.url_t) + '?ts=' + (+new Date()));
			item.querySelector('.photo').setAttribute('alt', photo.title);
			item.querySelector('.photo').setAttribute('title', photo.title);

			items.appendChild(item);
		});

		// Append the new cards to the list
		document.querySelector('.list').appendChild(items);
	};

	// Take a random image from the list and display it in background
	var displayCover = function(err, data) {
		if (err) throw new Error(err.message);

		var i = Math.floor(Math.random() * 50);

		var img = document.createElement('img');
		img.onload = function() {
			document.querySelector('.cover').style.backgroundImage = 'url(' + (data.photos.photo[i].url_o || data.photos.photo[i].url_l) + ')';
			document.querySelector('.cover').classList.add('ready');
		};
		img.src = data.photos.photo[i].url_l;
	};

	/*
	 * Bind events
	 */

	// Form submission
	document.querySelector('.search-form').addEventListener('submit', function(e) {
		e.preventDefault();

		emptyResults();

		document.querySelector('.loader').classList.add('active');
		nextPage = 2;

		flickr.search({
			text: encodeURIComponent(document.getElementById('search-form-q').value)
		}, replaceResults);

	}, true);

	// Image load
	document.querySelector('.list').addEventListener('load', function(e) {
		var card = e.target.parentNode.parentNode;
		card.classList.remove('flipped');
	}, true);

	// Infinite scroll
	window.addEventListener('scroll', function(e) {
		var fromBottom = document.body.offsetHeight - window.innerHeight - window.scrollY;
		if (hasScrollRequest || fromBottom > 700) return;

		hasScrollRequest = true;

		flickr.search({
			text: encodeURIComponent(document.getElementById('search-form-q').value),
			page: ++nextPage
		}, appendResults);

	}, true);

	// Load the cover image
	flickr.interesting({}, displayCover);

})();

5. created a lib folder and inserted the code to name the file flickr-search.js:

(function() {

	// Default configuration
	var defaultConfig = {
		api_uri: 'http://api.flickr.com/services/rest/',
		api_key: null
	};

	// Constructor
	window.Flickr = function(userConfig) {
		this.config = defaultConfig;
		for (var i in userConfig) {
			if (!userConfig.hasOwnProperty(i)) continue;
			this.config[i] = userConfig[i];
		}
	};

	// Temporary array for callbacks
	window.Flickr._jsonCallback = [];

	// Generic request method
	window.Flickr.prototype.request = function(params, callback) {
		var
			id   = Math.round(Math.random() * 1e9), // Generate a random ID for the request
			s    = document.createElement('script'),
			r    = document.getElementsByTagName('script')[0],
			args = [],
			i
		;

		// Append additional parameters to the request
		params.format = 'json';
		params.api_key = this.config.api_key;
		params.jsoncallback = 'Flickr._jsonCallback[' + id + ']';

		// "Serialize" parameters
		for (i in params) {
			if (!params.hasOwnProperty(i)) continue;
			args.push(i + '=' + params[i]);
		}

		// Store the callback in the temporary array
		window.Flickr._jsonCallback[id] = this._handleResponse.bind(this, id, callback, s);

		// Construct the request then append the script element in the page
		s.src = this.config.api_uri + '?' + args.join('&');
		r.parentNode.insertBefore(s, r);
	};

	// Generic response handler
	window.Flickr.prototype._handleResponse = function(id, callback, script, res) {
		// Cleaning
		delete window.Flickr._jsonCallback[id];
		script.parentNode.removeChild(script);

		// Handle errors
		if (res.stat === 'fail') return callback(res, null);

		// Return the request results to the callback
		callback(null, res);
	};

	// Photo search helper
	window.Flickr.prototype.search = function(params, callback) {
		params.method   = 'flickr.photos.search';
		params.extras   = params.extras || 'url_t,url_q';
		params.per_page = params.per_page || 50;
		this.request(params, callback);
	};

	// Interesting photos helper
	window.Flickr.prototype.interesting = function(params, callback) {
		params.method   = 'flickr.interestingness.getList';
		params.extras   = params.extras || 'url_l';
		params.per_page = params.per_page || 50;
		this.request(params, callback);
	};

	/*
	 * Helper boilerplate
	 *
	 *	window.Flickr.prototype.whatyouwant = function(params, callback) {
	 *		// Append additional parameters to the request
	 *		params.method = 'flickr.method'; // Mandatory, you MUST set a method
	 *		// Launch the request
	 *		this.request(params, callback);
	 *	};
	 */

})();

6. inserted both Javascript files into the HTML:

<script src="lib/flickr-search.js"></script>
		<script src="js/main.js"></script>
		
7. Added the Formedix logo to the Header of the HTML:

	<div data-role="header" data-position="inline">
			<div data-role="header">
			<img src="img/Formedix_logo.png" height="70px" style="float:centre;display:inline">
			
8. Added my name and Portfolio to the Footer:

<div data-role="footer" data-position="fixed">
       <p><a href="https://cwilson1993.000webhostapp.com/">©Craig Wilson</a></p>
    </div>