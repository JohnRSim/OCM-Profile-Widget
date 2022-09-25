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
	import OCERemoteJSON from './lib/api/oce_remoteJSON.js';
	import OCESocial from './lib/api/oce_social.js';

	//stores
	import { user as sUser } from './stores/user.js';
	import { people as sPeople } from './stores/people.js';
    import { each, prop_dev } from 'svelte/internal';

	const social = new OCESocial();
	const IDCS = new Auth();
	const OSN = new OCERemoteJSON();

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
		//login get profile info
		login();
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
				//sUser.updateOCEVal('id',connection.id);]
				
				//get Profile pic
				//this image is too scaled down god fr small profile images so use alternative req below
				//const [promiseProfilePic, abortProfilePic] = OSN.getMyProfilePic($sUser.session.oce, $sUser.sessionID);
				const [promiseProfilePic, abortProfilePic] = OSN.getProfilePictureDataUri([connection.user.id], $sUser.session.oce, $sUser.sessionID);
				
				//store data
				promiseProfilePic.then((profilePics) => {
					console.log('[getMyProfilePic]',profilePics);
					sPeople.updateProfileImg($sUser.profile.oce.id, profilePics[0].returnValue);
				});

				//store user profile info
				sUser.updateProfileInfo(connection.user, 'oce');

				loggedIn = true;
				//flip to show social widget
				flip = true;


				//get User Stats 
				const [promiseStats, abortStats] = social.getUserStats(connection.user.id, $sUser.session.oce);

				promiseStats.then((res) => {
					console.log('[getUserStats]',res);
					const stats = {
						followers: 0,
						following: 0,
						newConversations: 0,
					};
					res.items.forEach((v, i) => {
						if (v.name === 'FOLLOWERS_COUNT') {
							stats.followers = v.count;
						}
						if (v.name === 'FOLLOWING_COUNT') {
							stats.following = v.count;
						}
						if (v.name === 'NEW_CONVERSATIONS_COUNT') {
							stats.newConversations = v.count;
						}
						
					});
					//update stats
					sUser.updateStats(stats);
				});

				//get all custom user properties
				const [promiseProps, abortProps] = social.getAllUserProperties(connection.user.id, $sUser.session.oce);
				
				promiseProps.then((allUserProps) => {
					console.log('[OCE][Profile][getAllUserProperties]',allUserProps);
					
					//check if user has custom props
					if (allUserProps.items.some(prop => prop.name === 'Social_Enabled')) {
						//update props
						sUser.updateProps(allUserProps);
					//create custom props
					} else {
						//make sure to use string - boolean didn't work or error just no respone
						const userProps = [{
							name: 'Social_email',
							value: ''
						},{
							name: 'Social_facebook',
							value: ''
						},{
							name: 'Social_linkedIn',
							value: ''
						},{
							name: 'Social_twitter',
							value: ''
						},{
							name: 'Social_web',
							value: ''
						},{
							name: 'Social_Enabled',
							value: 'true'
						}];

						userProps.forEach((prop, i) => {
							sUser.updateProps(prop);
							createUserProperty(prop,connection.user.id);
						});
					}
				});
			});
		}).catch((err) => {
			console.error('IDCS OCE Auth Error: ', err);
			loginError = true;
		});
	}
	
	/**
	 * createUserProperty
	 * check if 'enter' key and then attempt to login
	 **/
	 function createUserProperty(prop, userID) {
		const [promise, abort] = social.createUserProperty(userID, prop, $sUser.session.oce, $sUser.sessionID);

		promise.then((createdProp) => {
			console.log('[OCE][Profile][CreatedProp]',prop);
		});
	}

	/**
	 * getProfileImage
	 * alternative example not used.
	*/
	function getProfilePic(userID) {
		console.log('[getProfilePic]',userID);
		const profilePicture = social.getProfilePic(userID,$sUser.session.oce);
		//console.log('[getProfilePic][profilePicture]',profilePicture);
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


	/**
	 * socialIco
	*/
	function socialIco(name) {
		let img = '';
		switch(name) {
			case 'Social_email':
				img = icoEmail;
			break;
			case 'Social_facebook':
				img = icoFb;
			break;
			case 'Social_linkedIn':
				img = icoLinkedin;
			break;
			case 'Social_twitter':
				img = icoTwitter;
			break;
			case 'Social_web':
				img = icoWeb;
			break;
		}
		return img;
	}
</script>


<!-- Wrapper -->
<section class="bitmapbytes-profileWidget">
	<!-- Widget -->
	<article>
		<!-- Container -->
		<figure>
			<!-- Profile Image -->
			{#if ($sPeople) && ($sPeople.profilePhoto) && ($sPeople.profilePhoto[72004])}
				<img src="{$sPeople.profilePhoto[72004].img}" alt="Profile Image" />
			{:else}
				<img src="" alt="Profile Image" />
			{/if}
			<!-- xProfile Image -->

			<!-- Copy -->
			<figcaption>
				<!-- User Info Panel -->
				{#if (($sUser) && ($sUser.profile) && ($sUser.profile.oce)) }
				<div class="userInfo">
					<!-- Display Name -->
					{#if ($sUser.profile.oce.displayName)}
						<h2>{$sUser.profile.oce.displayName}</h2>
					{/if}
					<!-- xDisplay Name -->

					<!-- Title -->
					{#if ($sUser.profile.oce.title)}
						<h3>{$sUser.profile.oce.title}</h3>
					{/if}
					<!-- xTitle -->
				</div>
				{/if}
				<!-- xUser Info Panel -->

				<!-- User Social Links Panel -->
				<nav class="socialLinks">
					{#if (($sUser) && ($sUser.props) && ($sUser.props.items))}
						{#each $sUser.props.items as prop}
							{#if ((prop.name) && (prop.name.startsWith('Social_') && (prop.name !== 'Social_Enabled') && (prop.value.length > 0)))}
								<a href="{prop.value}" title="{prop.name.replace(/Social_/g,'')}"><img src="{socialIco(prop.name)}" alt="{prop.name.replace(/Social_/g,'')}" /></a>
							{/if}
						{/each}
					{/if}
				</nav>
				<!-- xUser Social Links Panel -->
			</figcaption>
			<!-- xCopy -->
		</figure>
		<!-- xContainer -->
	</article>
	<!-- xWidget -->
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
