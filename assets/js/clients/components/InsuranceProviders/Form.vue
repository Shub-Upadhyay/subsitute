<template>
	<loading-indicator v-if="loading" class="my-5" />
	<validation-observer v-else v-bind="$attrs" ref="observer" v-slot="{ invalid }">
		<b-form @submit.prevent="save">
			<b-card no-body>
				<slot name="header"></slot>
				<b-card-body>
					<validation-provider
						vid="name"
						name="Name"
						:rules="{ required: true, min: 3, max: 50 }"
						v-slot="validationContext"
					>
						<b-form-group label="Name" label-for="name" label-cols-lg="4" label-cols-xl="3" class="mb-4">
							<b-form-input
								autofocus
								name="name"
								size="lg"
								type="text"
								v-model="entity.name"
								required
								placeholder="Required"
								:state="getValidationState(validationContext)"
								:disabled="saving"
							/>
							<b-form-invalid-feedback
								v-for="error in validationContext.errors"
								:key="error"
								v-text="error"
							/>
						</b-form-group>
					</validation-provider>

					<validation-provider
						vid="default_insurance_type_id"
						name="Type"
						:rules="{ required: true }"
						v-slot="validationContext"
					>
						<b-form-group
							label="Type"
							label-for="default_insurance_type_id"
							label-cols-lg="4"
							label-cols-xl="3"
						>
							<b-form-radio-group
								name="default_insurance_type_id"
								v-model="entity.default_insurance_type_id"
								:options="insuranceTypes"
								:disabled="saving || loadingInsuranceTypes"
								:state="getValidationState(validationContext)"
								value-field="id"
								text-field="name"
								required
							/>
							<b-form-invalid-feedback
								v-for="error in validationContext.errors"
								:key="error"
								v-text="error"
							/>
						</b-form-group>
					</validation-provider>

					<b-form-group label="Levels" label-for="entity.appeal_levels" label-cols-lg="4" label-cols-xl="3">
						<div v-if="entity.appeal_levels.length > 0">
							<div v-for="(appealLevel, index) in entity.appeal_levels" :key="index" class="mb-2">
								<b-card no-body>
									<b-card-header>
										<b-form-group class="mb-0">
											<b-input-group>
												<b-form-select
													v-model="appealLevel._joinData.appeal_level_id"
													:options="appealLevels"
													disabled
													value-field="id"
													text-field="name"
													required="required"
													placeholder="Level"
													aria-readonly="true"
												/>
												<b-input-group-prepend>
													<b-button
														variant="danger"
														@click="removeAppealLevel(appealLevel, index)"
														title="Remove this level"
													>
														<font-awesome-icon icon="times" fixed-width />
													</b-button>
												</b-input-group-prepend>
											</b-input-group>
										</b-form-group>
									</b-card-header>
									<b-card-body>
										<b-row>
											<b-col cols="12" xl="6">
												<b-form-group
													label="Name"
													description="This will override the default level of name"
													label-cols-lg="4"
												>
													<b-form-input
														name="label"
														type="text"
														v-model="appealLevel._joinData.label"
														:disabled="saving"
													/>
												</b-form-group>
											</b-col>
											<b-col cols="12" xl="6">
												<validation-provider
													vid="agency_id"
													name="Agency"
													:rules="{ required: false }"
													v-slot="validationContext"
												>
													<b-form-group
														label="Agency"
														label-for="agency_id"
														label-cols-lg="4"
													>
														<b-input-group>
															<b-form-select
																name="agency_id"
																v-model="appealLevel._joinData.agency_id"
																:disabled="saving"
																:options="agencies"
																value-field="id"
																text-field="name"
																:state="getValidationState(validationContext)"
															/>
															<template #append>
																<b-button
																	variant="primary"
																	@click="addingAgency = !addingAgency"
																	:active="addingAgency"
																>
																	<font-awesome-icon icon="plus" fixed-width />
																</b-button>
															</template>
														</b-input-group>
														<b-form-invalid-feedback
															v-for="error in validationContext.errors"
															:key="error"
															v-text="error"
														/>
													</b-form-group>
												</validation-provider>
											</b-col>
											<b-col cols="12" xl="6">
												<validation-provider
													vid="days_to_respond"
													name="Days to respond"
													:rules="{ required: true, min: 0, max: 365 }"
													v-slot="validationContext"
												>
													<b-form-group
														label="Days To Respond"
														label-for="days_to_respond"
														label-cols-lg="4"
														description="The amount of days from before a response is due"
													>
														<b-form-input
															name="days_to_respond"
															type="number"
															step="1"
															min="0"
															max="365"
															default="30"
															v-model="appealLevel._joinData.days_to_respond"
															:disabled="saving"
															:state="getValidationState(validationContext)"
															required="required"
														/>
														<b-form-invalid-feedback
															v-for="error in validationContext.errors"
															:key="error"
															v-text="error"
														/>
													</b-form-group>
												</validation-provider>
											</b-col>
											<b-col cols="12" xl="6">
												<validation-provider
													vid="max_days"
													name="Maximum Response Days"
													:rules="{ required: true, min: 0, max: 365 }"
													v-slot="validationContext"
												>
													<b-form-group
														label="Maximum Response Days"
														label-for="max_days"
														label-cols-lg="4"
														description="The maximum days before a return response is expected"
													>
														<b-form-input
															name="max_days"
															type="number"
															step="1"
															min="0"
															max="365"
															default="30"
															v-model="appealLevel._joinData.max_days"
															:disabled="saving"
															:state="getValidationState(validationContext)"
															required="required"
														/>
														<b-form-invalid-feedback
															v-for="error in validationContext.errors"
															:key="error"
															v-text="error"
														/>
													</b-form-group>
												</validation-provider>
											</b-col>
										</b-row>

										<b-modal
											v-model="addingAgency"
											size="xl"
											hide-header
											hide-footer
											body-class="p-0"
											content-class="p-0"
										>
											<agency-form
												@cancel="addingAgency = false"
												@saved="addedNewAgency($event, appealLevel, index)"
											>
												<template #header>
													<b-card-header>
														<div class="d-flex justify-content-between align-items-center">
															<span class="font-weight-bold">Add New Agency</span>
															<b-button
																variant="secondary"
																size="sm"
																@click="addingAgency = false"
																title="Cancel"
																class="mb-0"
															>
																<font-awesome-icon
																	icon="remove"
																	fixed-width
																	class="my-0 py-0"
																/>
															</b-button>
														</div>
													</b-card-header>
												</template>
											</agency-form>
										</b-modal>
									</b-card-body>
								</b-card>
							</div>

							<b-button
								block
								size="lg"
								variant="secondary"
								@click="addAppealLevel"
								:disabled="!canAddAppealLevel"
								class="my-3"
							>
								<font-awesome-icon icon="plus" fixed-width />
								<span>Add Level</span>
							</b-button>
						</div>
						<div v-else>
							<empty-result icon="gavel">
								No appeal levels
								<template #content>
									<p>Add appeal levels to restrict or rename opportunities.</p>

									<b-button
										size="lg"
										variant="primary"
										@click="addAppealLevel"
										:disabled="!canAddAppealLevel"
										class="my-2"
									>
										<font-awesome-icon icon="plus" fixed-width />
										<span>Add Level</span>
									</b-button>
								</template>
							</empty-result>
						</div>
					</b-form-group>
				</b-card-body>

				<b-card-body>
					<h6 class="text-muted">Optional</h6>

					<b-card no-body class="mb-0">
						<b-card-header header-tag="header" role="tab" class="p-0">
							<b-button
								block
								v-b-toggle.collapseAdditional
								variant="light"
								role="tab"
								class="text-left px-4 py-3 m-0"
								>Additional Details</b-button
							>
						</b-card-header>
						<b-collapse id="collapseAdditional" role="tabpanel">
							<b-card-body>
								<b-form-group
									label="Active"
									label-for="active"
									label-cols-lg="4"
									description="Inactive providers will not show up in dropdown lists."
								>
									<b-form-checkbox name="active" v-model="entity.active">Active</b-form-checkbox>
								</b-form-group>

								<b-form-group label="Available Types" label-for="insurance_type_ids" label-cols-lg="4">
									<b-form-checkbox-group
										stacked
										name="insurance_type_ids"
										v-model="entity.insurance_types._ids"
										:options="insuranceTypes"
										:disabled="saving || loadingInsuranceTypes"
										value-field="id"
										text-field="name"
									/>
								</b-form-group>
							</b-card-body>
						</b-collapse>

						<b-card-header header-tag="header" role="tab" class="p-0">
							<b-button
								block
								v-b-toggle.collapseAddress
								variant="light"
								role="tab"
								class="text-left px-4 py-3 m-0"
								>Address</b-button
							>
						</b-card-header>
						<b-collapse id="collapseAddress" role="tabpanel">
							<b-card-body>
								<b-form-group label="Address" label-for="street_address_1" label-cols-lg="4">
									<validation-provider
										vid="street_address_1"
										name="Street Address"
										:rules="{ required: false, max: 50 }"
										v-slot="validationContext"
									>
										<b-form-input
											name="street_address_1"
											type="text"
											v-model="entity.street_address_1"
											placeholder=""
											class="rounded-b-0"
											:state="getValidationState(validationContext)"
											:disabled="saving"
										/>
										<b-form-invalid-feedback
											v-for="error in validationContext.errors"
											:key="error"
											v-text="error"
										/>
									</validation-provider>
									<validation-provider
										vid="street_address_2"
										name="Street Address (Continued)"
										:rules="{ required: false, max: 50 }"
										v-slot="validationContext"
									>
										<b-form-input
											name="street_address_2"
											type="text"
											v-model="entity.street_address_2"
											placeholder=""
											class="rounded-t-0"
											:state="getValidationState(validationContext)"
											:disabled="saving"
										/>
										<b-form-invalid-feedback
											v-for="error in validationContext.errors"
											:key="error"
											v-text="error"
										/>
									</validation-provider>
								</b-form-group>

								<validation-provider
									vid="city"
									name="City"
									:rules="{ required: false, max: 50 }"
									v-slot="validationContext"
								>
									<b-form-group label="City" label-for="city" label-cols-lg="4">
										<b-form-input
											name="city"
											type="text"
											v-model="entity.city"
											:state="getValidationState(validationContext)"
											:disabled="saving"
										/>
										<b-form-invalid-feedback
											v-for="error in validationContext.errors"
											:key="error"
											v-text="error"
										/>
									</b-form-group>
								</validation-provider>

								<validation-provider
									vid="state"
									name="State"
									:rules="{ required: false, max: 2 }"
									v-slot="validationContext"
								>
									<b-form-group label="State" label-for="state" label-cols-lg="4">
										<b-form-select
											name="state"
											v-model="entity.state"
											:options="states"
											value-field="abbreviation"
											text-field="name"
											:state="getValidationState(validationContext)"
											:disabled="saving"
										>
											<template #first>
												<option :value="null" />
											</template>
										</b-form-select>
										<b-form-invalid-feedback
											v-for="error in validationContext.errors"
											:key="error"
											v-text="error"
										/>
									</b-form-group>
								</validation-provider>

								<validation-provider
									vid="zip"
									name="Zip"
									:rules="{ required: false, max: 20, alpha_num: true }"
									v-slot="validationContext"
								>
									<b-form-group label="Zip" label-for="zip" label-cols-lg="4">
										<b-form-input
											name="zip"
											type="text"
											v-model="entity.zip"
											:state="getValidationState(validationContext)"
											:disabled="saving"
										/>
										<b-form-invalid-feedback
											v-for="error in validationContext.errors"
											:key="error"
											v-text="error"
										/>
									</b-form-group>
								</validation-provider>
							</b-card-body>
						</b-collapse>

						<b-card-header header-tag="header" role="tab" class="p-0">
							<b-button
								block
								v-b-toggle.collapseContact
								variant="light"
								role="tab"
								class="text-left px-4 py-3 m-0"
								>Contact</b-button
							>
						</b-card-header>
						<b-collapse id="collapseContact" role="tabpanel">
							<b-card-body>
								<validation-provider
									vid="phone"
									name="Phone"
									:rules="{ required: false }"
									v-slot="validationContext"
								>
									<b-form-group label="Phone" label-for="phone" label-cols-lg="4">
										<b-form-input
											name="phone"
											type="text"
											v-model="entity.phone"
											v-mask="'(###) ###-####'"
											:state="getValidationState(validationContext)"
											:disabled="saving"
										/>
										<b-form-invalid-feedback
											v-for="error in validationContext.errors"
											:key="error"
											v-text="error"
										/>
									</b-form-group>
								</validation-provider>

								<validation-provider
									vid="fax"
									name="Fax"
									:rules="{ required: false }"
									v-slot="validationContext"
								>
									<b-form-group label="Fax" label-for="fax" label-cols-lg="4">
										<b-form-input
											name="fax"
											type="text"
											v-model="entity.fax"
											v-mask="'(###) ###-####'"
											:state="getValidationState(validationContext)"
											:disabled="saving"
										/>
										<b-form-invalid-feedback
											v-for="error in validationContext.errors"
											:key="error"
											v-text="error"
										/>
									</b-form-group>
								</validation-provider>

								<validation-provider
									vid="email"
									name="Email"
									:rules="{ required: false, email: true }"
									v-slot="validationContext"
								>
									<b-form-group label="Email" label-for="email" label-cols-lg="4">
										<b-form-input
											name="email"
											type="email"
											v-model="entity.email"
											:state="getValidationState(validationContext)"
											:disabled="saving"
										/>
										<b-form-invalid-feedback
											v-for="error in validationContext.errors"
											:key="error"
											v-text="error"
										/>
									</b-form-group>
								</validation-provider>
							</b-card-body>
						</b-collapse>
					</b-card>
				</b-card-body>

				<b-card-footer>
					<b-row>
						<b-col cols="12" md="6" xl="4" class="mb-4 mb-md-0">
							<b-button block variant="light" type="button" @click.stop="cancel">Cancel</b-button>
						</b-col>
						<b-col cols="12" md="6" offset-xl="4" xl="4" class="mb-2 mb-md-0">
							<b-button block variant="primary" type="submit" :disabled="saving || invalid">
								<font-awesome-icon icon="circle-notch" v-if="saving" spin fixed-width />
								<span>Save</span>
							</b-button>
						</b-col>
					</b-row>
				</b-card-footer>
			</b-card>
		</b-form>
	</validation-observer>
</template>

<script type="text/javascript">
import { mapGetters } from "vuex";
import { formatErrors, getValidationState } from "@/validation";
import AgencyForm from "@/clients/components/Agencies/Form.vue";

export default {
	name: "InsuranceProviderForm",
	components: {
		AgencyForm,
	},
	props: {
		id: {
			default: null,
		},
	},
	data() {
		return {
			loading: true,
			saving: false,
			entity: {
				id: this.id,
				default_insurance_type_id: null,
				name: "",
				phone: "",
				fax: "",
				active: true,
				street_address_1: null,
				street_address_2: null,
				city: null,
				state: null,
				zip: null,
				insurance_types: {
					_ids: [],
				},
				appeal_levels: [
					// {} Join table entity
				],
			},
			addingAgency: false,
		};
	},
	computed: {
		canAddAppealLevel() {
			if (this.loadingAppealLevels) {
				return false;
			}

			const currentAppealLevels = this.entity?.appeal_levels?.length ?? 0;
			const totalAppealLevels = this.appealLevels?.length ?? 0;

			if (currentAppealLevels >= totalAppealLevels) {
				return false;
			}

			return true;
		},
		availableAppealLevels() {
			return this.appealLevels.filter((appealLevel) => {
				return !this.entity.appeal_levels.some((entity) => {
					return (
						entity.appeal_level_id == appealLevel.id || entity._joinData.appeal_level_id == appealLevel.id
					);
				});
			});
		},
		...mapGetters({
			states: "states/states",
			agencies: "agencies/active",
			loadingAgencies: "agencies/loadingActive",
			appealLevels: "appealLevels/all",
			loadingAppealLevels: "appealLevels/loadingAll",
			insuranceTypes: "insuranceTypes/all",
			loadingInsuranceTypes: "insuranceTypes/loadingAll",
		}),
	},
	mounted() {
		if (this.id) {
			this.refresh();
		} else {
			this.loading = false;
		}
	},
	methods: {
		getValidationState,
		cancel() {
			this.$emit("cancel");
		},
		async refresh() {
			try {
				this.loading = true;

				const response = await this.$store.dispatch("insuranceProviders/get", {
					id: this.id,
				});

				this.entity = response;
				this.entity.insurance_types = {
					_ids: response.insurance_types.map((insuranceType) => insuranceType.id) ?? [],
				};

				this.$emit("loaded", response);
			} catch (e) {
				this.$store.dispatch("apiError", {
					error: e,
					message: "Error getting insurance provider details",
				});
			} finally {
				this.loading = false;
			}
		},
		async save() {
			try {
				this.saving = true;

				const request = Object.assign({}, this.entity);
				const response = await this.$store.dispatch("insuranceProviders/save", request);

				this.$emit("saved", response);
				this.$emit("update:id", response.id);
			} catch (e) {
				if (e.response.data.errors) {
					this.$refs.observer.setErrors(formatErrors(e.response.data.errors));
				}

				this.$store.dispatch("apiError", {
					error: e,
					title: "Save Failed",
					message: "Error saving insurance provider details. Please check for errors.",
					variant: "warning",
				});
			} finally {
				this.saving = false;
				this.$store.dispatch("insuranceProviders/getActive");
				this.$store.dispatch("insuranceProviders/getAll");
			}
		},
		addAppealLevel() {
			const nextId = this.availableAppealLevels[0]?.id ?? null;

			this.entity.appeal_levels.push({
				id: nextId,
				_joinData: {
					id: null,
					appeal_level_id: nextId,
					label: "",
					days_to_respond: 30,
					max_days: 30,
					agency_id: null,
				},
			});
		},
		removeAppealLevel(appealLevel, index) {
			this.entity.appeal_levels.splice(index, 1);
		},
		async addedNewAgency(agency, appealLevel = null, index = null) {
			this.$store.dispatch("agencies/getActive");
			this.addingAgency = false;

			// Set appeal level's agency ID to newly created agency
			if (index !== null && index !== undefined && appealLevel && appealLevel._joinData) {
				this.entity.appeal_levels[index]._joinData.agency_id = agency.id;
			}
		},
	},
	watch: {
		entity: {
			deep: true,
			handler(val) {
				const insuranceTypeId = this.entity.default_insurance_type_id ?? null;
				const exists = this.entity.insurance_types?._ids?.includes(insuranceTypeId) ?? null;

				// Ensure the default type is selected
				if (insuranceTypeId !== null && exists == false) {
					this.entity.insurance_types._ids.push(insuranceTypeId);
				}
			},
		},
		insuranceTypes: {
			immediate: true,
			deep: true,
			handler(val) {
				const defaultInsuranceType = val.find((type) => type.name == "Private");
				if (defaultInsuranceType !== undefined && this.entity.default_insurance_type_id == null) {
					this.entity.default_insurance_type_id = defaultInsuranceType?.id ?? null;
				}
			},
		},
	},
};
</script>
