<svelte:options tag="ocm-profile-widget" />

<script>
	//svelte
	import { onMount } from 'svelte';
	//conf
	import OCMConf from './lib/config/oce.json';

	//assets
	import icoEmail from './assets/ico_email.svg';
	import icoFb from './assets/ico_fb.svg';
	import icoLinkedin from './assets/ico_linkedin.svg';
	import icoTwitter from './assets/ico_twitter.svg';
	import icoWeb from './assets/ico_web.svg';
	
	//api
	import Auth from './lib/api/auth.js';
	import OCESocial from './lib/api/oce_social.js';

	//stores
	import { user as sUser } from './stores/user.js';

	const social = new OCESocial();
	const IDCS = new Auth();

	//globals
	let isMounted = false;
	let userFieldActive = false;
	let passFieldActive = false;
	let flip = false;
	let loginError = false;
	let loggedIn = false;

	let userProfilePhotos = {};

	//auth
	let username = '';
	let password = '';

	onMount(() => {
		isMounted = true;
	});

	/**
	 * login
	 **/
	 function login() {
		//get OCE session token
		const [promise, abort] = IDCS.login(username, password);

		promise.then((oce) => {
			console.log('[OCE][Auth][Logged In][Token]',oce);

			//check if IDCS errored - 
			if (typeof(oce.error) !== 'undefined') {
				console.error('IDCS OCE Auth Error: ', oce.error);
				loginError = true;
				//loginError();
				return;
			}

			//update session store
			sUser.updateSession(oce.access_token,'oce');

			//set sessionID if null set to active login to reuse throughout journey
			let sessionID = ($sUser.sessionID)? $sUser.sessionID: Date.now();

			//get conversation connection info
			const [promise, abort] = social.getConnectionInfo($sUser.session.oce, $sUser.useSessionRequests, sessionID);

			promise.then((connection) => {
				console.log('[connection]',connection);
				
				//set sessionID
				sessionID = ($sUser.useSessionRequests) ? sessionID : connection.apiRandomID;
				sUser.updateConnectionInfo(connection.languageLocale, connection.apiVersion, sessionID);
				//sUser.updateOCEVal('id',connection.id);

				//store user profile info
				sUser.updateProfileInfo(connection.user, 'oce');

				loggedIn = true;
				//flip to show social widget
				flip = true;


			});
		}).catch((err) => {
			console.error('IDCS OCE Auth Error: ', err);
			loginError = true;
		});
	}

	/**
	 * setProperties
	 * Used for CustomElement
	 * @param node
	 * @param properties
	 */
	function setProperties(node, properties) {
		const applyProperties = () => {
			Object.entries(properties).forEach(([k, v]) => {
				node[k] = v;
			});
		};

		applyProperties();

		return {
			update(updatedProperties) {
				properties = updatedProperties;
				applyProperties();
			}
		};
	}

</script>


<!-- Wrapper -->
<section class="bitmapbytes-profileWidget">
	<article>
		<figure>
			<img src="#" alt="Profile Image" />
			<figcaption>
				<div class="userInfo">
					<h2>JS Nashorn</h2>
					<h3>Experimental Engineer</h3>
				</div>
				<nav class="socialLinks">
					<a href="#" title="email"><img src="{icoEmail}" alt="email" /></a>
					<a href="#" title="Facebook"><img src="{icoFb}" alt="Facebook" /></a>
					<a href="#" title="LinkedIn"><img src="{icoLinkedin}" alt="LinkedIn" /></a>
					<a href="#" title="Twitter"><img src="{icoTwitter}" alt="Twitter" /></a>
					<a href="#" title="Website"><img src="{icoWeb}" alt="Website" /></a>
				</nav>
			</figcaption>
		</figure>
	</article>
</section>
<!-- xWrapper -->

	
<style>
	section {
		background:#F3F4F6;
		padding:20px;
	}

	article {
		background: #FAFBFD;
		border-radius: 10px;
		box-shadow:0px 2px 4px  0px rgba(0,0,0,0.15);
		overflow: hidden;
		margin:10px;
		color:#090F1B;
	}

	figure {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding:10px;
		margin:10px;
	}

	figure > img {
		background:#000;
		display: block;
		width:140px;
		height:140px;
		border-radius: 50%;
		overflow: hidden;
		border:0px;
		background-size: cover;
		margin:0px 20px 20px 20px;
	}
	.userInfo {
		margin-bottom:20px;
	}
	.socialLinks {
		display: flex;
		border-top:solid 2px #2A2B2D;
		padding-top:10px;
		justify-content: center;
	}
	
	figcaption a {
		flex:1;
		max-width:26px;
		display: flex;
		margin:0px 4px;
		padding:4px;
		border-radius: 4px;;
	}
	figcaption a:hover {
		background: #F3F4F6;;
	}
	figcaption a:hover img {
		opacity:1;
	}
	figcaption img {
		width:100%;
		opacity:0.75;
	}

	h2 {
		text-align: center;
		color:#090F1B;
		margin:0px 0px 6px;
		padding:0px;
	}

	h3 {
		text-align: center;
		color:#818589;
		margin:0px;
		padding:0px;
		font-weight: 500;
		font-size:1em;
	}
</style>
