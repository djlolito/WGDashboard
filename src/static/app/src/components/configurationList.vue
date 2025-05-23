<script>
import {WireguardConfigurationsStore} from "@/stores/WireguardConfigurationsStore.js";
import ConfigurationCard from "@/components/configurationListComponents/configurationCard.vue";
import LocaleText from "@/components/text/localeText.vue";
import SystemStatus from "@/components/systemStatusComponents/systemStatusWidget.vue";
import {GetLocale} from "@/utilities/locale.js";

export default {
	name: "configurationList",
	components: {SystemStatus, LocaleText, ConfigurationCard},
	async setup(){
		const wireguardConfigurationsStore = WireguardConfigurationsStore();
		return {wireguardConfigurationsStore}
	},
	data(){
		return {
			configurationLoaded: false,
			sort: {
				Name: GetLocale("Name"),
				Status: GetLocale("Status"),
				'DataUsage.Total': GetLocale("Total Usage")
			},
			currentSort: {
				key: "Name",
				order: "asc"
			},
			currentDisplay: "List",
			searchKey: ""
		}
	},
	async mounted() {
		if (!window.localStorage.getItem('ConfigurationListSort')){
			window.localStorage.setItem('ConfigurationListSort', JSON.stringify(this.currentSort))
		}else{
			this.currentSort = JSON.parse(window.localStorage.getItem('ConfigurationListSort'))
		}

		if (!window.localStorage.getItem('ConfigurationListDisplay')){
			window.localStorage.setItem('ConfigurationListDisplay', this.currentDisplay)
		}else{
			this.currentDisplay = window.localStorage.getItem('ConfigurationListDisplay')
		}
		
		
		await this.wireguardConfigurationsStore.getConfigurations();
		this.configurationLoaded = true;
		
		this.wireguardConfigurationsStore.ConfigurationListInterval = setInterval(() => {
			this.wireguardConfigurationsStore.getConfigurations()
		}, 10000)
	},
	beforeUnmount() {
		clearInterval(this.wireguardConfigurationsStore.ConfigurationListInterval)
	},
	computed: {
		configurations(){
			return [...this.wireguardConfigurationsStore.Configurations]
				.filter(x => x.Name.toLowerCase().includes(this.searchKey) || x.PublicKey.includes(this.searchKey) || !this.searchKey)
				.sort((a, b) => {
				if (this.currentSort.order === 'desc') {
					return this.dotNotation(a, this.currentSort.key) < this.dotNotation(b, this.currentSort.key) ? 
						1 : this.dotNotation(a, this.currentSort.key) > this.dotNotation(b, this.currentSort.key) ? -1 : 0;
				} else {
					return this.dotNotation(a, this.currentSort.key) > this.dotNotation(b, this.currentSort.key) ? 
						1 : this.dotNotation(a, this.currentSort.key) < this.dotNotation(b, this.currentSort.key) ? -1 : 0;
				}
			})
		}
	},
	methods: {
		dotNotation(object, dotNotation){
			let result = dotNotation.split('.').reduce((o, key) => o && o[key], object)
			if (typeof result === "string"){
				return result.toLowerCase()
			}
			return result
		},
		updateSort(key){
			if (this.currentSort.key === key){
				if (this.currentSort.order === 'asc') this.currentSort.order = 'desc'
				else this.currentSort.order = 'asc'
			}else{
				this.currentSort.key = key
			}
			window.localStorage.setItem('ConfigurationListSort', JSON.stringify(this.currentSort))
		},
		updateDisplay(key){
			if (this.currentDisplay !== key){
				this.currentDisplay = key
				window.localStorage.setItem('ConfigurationListDisplay', this.currentDisplay)
			}
		}
	}
}
</script>

<template>
	<div class="mt-md-5 mt-3">
		<div class="container-fluid">
			<SystemStatus></SystemStatus>
			<div class="d-flex mb-4 configurationListTitle align-items-md-center gap-2 flex-column flex-md-row">
				<h2 class="text-body d-flex mb-0">
					<LocaleText t="WireGuard Configurations"></LocaleText>
				</h2>
				<RouterLink to="/new_configuration"
				            class="ms-md-auto py-2 text-decoration-none btn text-primary-emphasis bg-primary-subtle rounded-3 border-1 border-primary-subtle">
					<i class="bi bi-plus-circle me-2"></i><LocaleText t="Configuration"></LocaleText>
				</RouterLink>
				<RouterLink to="/restore_configuration"
				            class="py-2 text-decoration-none btn text-primary-emphasis bg-primary-subtle rounded-3 border-1 border-primary-subtle">
					<i class="bi bi-clock-history me-2"></i><LocaleText t="Restore"></LocaleText>
				</RouterLink>
				
			</div>
			<Transition name="fade">
				<div class="text-body filter mb-3 d-flex gap-2 flex-column flex-md-row" v-if="this.configurationLoaded">
					<div class="d-flex align-items-center gap-3 align-items-center mb-3 mb-md-0">
						<small class="text-muted">
							<LocaleText t="Sort By"></LocaleText>
						</small>
						<div class="d-flex ms-auto ms-lg-0">
							<a role="button"
							   @click="updateSort(sv)"
							   :class="{'bg-primary-subtle text-primary-emphasis': this.currentSort.key === sv}"
							   class="px-2 py-1 rounded-3" v-for="(s, sv) in this.sort">
								<small>
									<i class="bi me-2"
									   :class="[this.currentSort.order === 'asc' ? 'bi-sort-up' : 'bi-sort-down']"
									   v-if="this.currentSort.key === sv"></i>{{s}}
								</small>
							</a>
						</div>
					</div>
					<div class="align-items-center gap-3 align-items-center mb-3 mb-md-0 d-none d-lg-flex ">
						<small class="text-muted">
							<LocaleText t="Display as"></LocaleText>
						</small>
						<div class="d-flex ms-auto ms-lg-0">
							<a role="button"
							   @click="updateDisplay(x.name)"
							   v-for="x in [{name: 'List', key: 'list'}, {name: 'Grid', key: 'grid'}]"
							   :class="{'bg-primary-subtle text-primary-emphasis': this.currentDisplay === x.name}"
							   class="px-2 py-1 rounded-3">
								<small>
									<i class="bi me-2" :class="'bi-' + x.key"></i> <LocaleText :t="x.name"></LocaleText>
								</small>
							</a>

						</div>
					</div>
					<div class="d-flex align-items-center ms-md-auto">
						<label for="configurationSearch" class="text-muted">
							<i class="bi bi-search me-2"></i>
						</label>
						<input class="form-control form-control-sm rounded-3"
						       v-model="this.searchKey"
						       id="configurationSearch">
					</div>
				</div>
			</Transition>
			
			<div class="row g-3 mb-2">
				<TransitionGroup name="fade">
					<p class="text-muted col-12"
					   key="noConfiguration"
					   v-if="this.configurationLoaded && this.wireguardConfigurationsStore.Configurations.length === 0">
						<LocaleText t="You don't have any WireGuard configurations yet. Please check the configuration folder or change it in Settings. By default the folder is /etc/wireguard."></LocaleText>
					</p>
					<ConfigurationCard
						:display="this.currentDisplay"
						v-for="(c, index) in configurations"
						:delay="index*0.03 + 's'"
						v-else-if="this.configurationLoaded"
						:key="c.Name" :c="c"></ConfigurationCard>
				</TransitionGroup>
			</div>
			
		</div>
	</div>
	
</template>

<style scoped>
.filter a{
	text-decoration: none;
}
</style>