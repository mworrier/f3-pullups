<template>
	<MABAForm>
		<section class="subsection description">
			<h1>Welcome to F3 JeffCo's Pullups February Challenge!</h1>

			<p><strong>The challenge:</strong> @Obiwan challenges you to 1,000 pullups in February. There are 20 weekdays in the month.  You can do 50 every weekday, or you can
				bank a bunch and
				take days off. Use weight, don't use weight.  Whatever you do, just be sure to Grease the Groove, baby! </p>

			<p v-if="userCanRegister">Sign up below.</p>
			<p v-else><strong>Registration is closed. Come back and join us for the next challenge!</strong></p>

		</section>

		<section class="subsection">
			<h2>Region</h2>
			<p v-if="userCanRegister"><em>Choose NONE if you are not an F3 member.</em></p>
			<select name="regions"
					v-if="regions.length"
					:value="selectedRegion"
					@change="onRegionChange"
			>
				<option v-for="region in regions"
						:key="region"
						:value="region"
				>{{ region }}
				</option>
			</select>

			<h2>F3 Name</h2>
			<p v-if="userCanRegister"><em>Use your first and last name if you are not an F3 member.</em></p>
			<div class="f3-name-container" v-if="userCanRegister">
				<div>
					<p v-if="hims.length">
						<label for="sign-up-option">
							<input type="radio"
								   id="sign-up-option"
								   name="himStatus"
								   :checked="canCreateHim"
								   @input="onHimStatusChange( `NEW` )"
							/>
							Sign up
						</label>
					</p>
					<p v-else-if="!hims.length">There are no previous registrants for the selected
						region/AO. Be the first!</p>
					<input type="text"
						   placeholder="e.g., dredd"
						   @input="onNameInput"
						   :class="nameFieldClasses"
						   :disabled="!canCreateHim"
					/>
					<h3>Email</h3>
					<input type="text"
						   placeholder="e.g., dredd@f3nation.com"
						   @input="onEmailInput"
						   :class="emailFieldClasses"
						   :disabled="!canCreateHim"
					/>
				</div>
				<div v-if="hims.length">
					<h3>- OR -</h3>
				</div>
				<div v-if="hims.length">
					<p>
						<label for="new-burpees-option">
							<input type="radio"
								   id="new-burpees-option"
								   name="himStatus"
								   :checked="canSelectHim"
								   @input="onHimStatusChange( `EXISTING` )"
							/>
							Input new pullups
						</label>
					</p>
					<select name="hims"
							:key="hims[ 0 ]?.him_id"
							:value="selectedHimId"
							:disabled="!canSelectHim"
							@change="onAOHimChange"
					>
						<option v-for="him in hims"
								:key="him.him_id"
								:value="him.him_id"
								:selected="him.him_id === selectedHimId"
						>{{ him.f3_name }}
						</option>
					</select>
				</div>
			</div>
			<div class="f3-name-container" v-else>
				<div v-if="hims.length">
					<select name="hims"
							:key="hims[ 0 ]?.him_id"
							:value="selectedHimId"
							:disabled="!canSelectHim"
							@change="onAOHimChange"
					>
						<option v-for="him in hims"
								:key="him.him_id"
								:value="him.him_id"
								:selected="him.him_id === selectedHimId"
						>{{ him.f3_name }}
						</option>
					</select>
				</div>
				<p v-else>No PAX have registered for this region.</p>
			</div>
		</section>

		<section class="subsection" v-if="mergedBurpees.length">
			<h2 class="burpees-heading">
				<span v-if="himName">Pullups for {{ himName }}</span>
				<span v-else>{{ burpeeYear }} Burpees</span>
				<span>Total: {{ totalBurpees }}</span>
			</h2>
			<Burpees :burpees="mergedBurpees"
					 :disabled="!userCanRecordBurpees"
					 @change="onBurpeesChanged"
			/>
			<p class="centered">
				<a class="daily-stats-link" href="/stats">Daily Stats</a>
			</p>
		</section>

		<section class="subsection spread buttons" v-if="userCanRecordBurpees">
			<div class="buttons--left">
				<button @click="onSubmitForm">Submit</button>
			</div>
			<div class="buttons--right">
				<button @click="onResetBurpees" v-if="hasModifiedBurpees">Reset burpees ONLY</button>
				<button @click="onResetForm">Reset form</button>
			</div>
		</section>
	</MABAForm>
</template>

<script>
import { mapGetters, mapMutations, mapActions } from "vuex";
import MABAForm from "./MABAForm";
import Burpees from "./Burpees";

export default {
	name: "SignupPage",
	components: {
		MABAForm,
		Burpees,
	},
	computed: {
		...mapGetters( "signupPage", [
			"userCanRegister",
			"userCanRecordBurpees",
			"burpeeYear",
			"validation",
			"regions",
			"hims",
			"hasEnteredName",
			"hasEnteredEmail",
			"mergedBurpees",
			"canSelectHim",
			"canCreateHim",
			"himStatus",
			"selectedHimId",
			"totalBurpees",
			"selectedRegion",
			"himName",
			"hasModifiedBurpees",
		] ),
		nameFieldClasses() {
			const { name: nameHasError, } = this.validation;
			if ( !this.hasEnteredName ) {
				return "";
			}
			return nameHasError ? "invalid" : "";
		},
		emailFieldClasses() {
			const { email: emailHasError, } = this.validation;
			if ( !this.hasEnteredEmail ) {
				return "";
			}
			return emailHasError ? "invalid" : "";
		},
	},
	methods: {
		...mapMutations( "signupPage", [
			"changeName",
			"changeEmail",
			"changeBurpeeCount",
			"resetBurpees",
		] ),
		...mapActions( "signupPage", [
			"changeRegion",
			"changeHim",
			"save",
			"changeHimStatus",
			"resetStore",
		] ),
		onRegionChange( e ) {
			const { target } = e;
			const { options, selectedIndex } = target;
			const region = options[ selectedIndex ].value;
			this.changeRegion( region );
		},
		onAOHimChange( e ) {
			const { target } = e;
			const { options, selectedIndex } = target;
			const himId = options[ selectedIndex ].value;
			this.changeHim( himId );
		},
		onNameInput( e ) {
			this.changeName( e.target.value );
		},
		onEmailInput( e ) {
			this.changeEmail( e.target.value );
		},
		onHimStatusChange( status ) {
			this.changeHimStatus( status );
		},
		onBurpeesChanged( e ) {
			const { date, newCount } = e;
			this.changeBurpeeCount( {
				date,
				count: newCount,
			} );
		},
		onSubmitForm( e ) {
			e.preventDefault();
			this.save();
		},
		onResetBurpees() {
			this.resetBurpees();
		},
		onResetForm() {
			this.resetStore();
		},
	}
};
</script>

<style scoped lang="scss">
@import "../assets/styles/styles";

.f3-name-container {
	width: 100%;
	display: flex;
	flex-direction: column;

	@include media-tablet() {
		flex-direction: row;
	}

	div {
		display: flex;
		flex-direction: column;
		justify-content: flex-start;
		align-content: center;
	}

	input, select {
		width: auto;
	}

	div:first-child, div:last-child {
		flex: 3;
	}

	div:nth-child(2) {
		flex: 1;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-content: center;
		font-weight: bold;
		font-style: italic;
	}
}

.burpees-heading {
	display: flex;
	flex-direction: column;
	justify-content: space-between;

	@include media-tablet() {
		flex-direction: row;
	}
}

.buttons--left, .buttons--right {
	display: flex;
	flex-direction: column;

	button {
		margin-top: 1rem;
	}

	@include media-tablet() {
		flex-direction: row;

		button {
			margin-top: 0;
			margin-right: 1rem;

			&:last-of-type {
				margin-right: 0;
			}
		}
	}
}

.daily-stats-link {
	display: inline-block;
	margin-top: 2rem;
}

.promo-video {
	padding: 1.5rem 0;
	background-color: black;
	border-radius: 0.5rem;

	iframe {
		width: 100%;

		@include media-tablet() {
			width: 90%;
		}
	}
}
</style>
