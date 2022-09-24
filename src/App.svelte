<svelte:options tag="ocm-social-widget" />

<script>
	//svelte
	import { onMount } from 'svelte';
	//conf
	import OCMConf from './lib/config/oce.json';

	//assets
	
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
</section>
<!-- xWrapper -->

<style>
</style>
