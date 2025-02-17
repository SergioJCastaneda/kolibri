<template>

  <CoreBase
    :immersivePage="false"
    :appBarTitle="appBarTitle"
    :authorized="userIsAuthorized"
    authorizedRole="adminOrCoach"
    :showSubNav="false"
  >

    <template #sub-nav>
      <TopNavbar />
    </template>

    <KPageContainer>

      <p>
        <KRouterLink
          v-if="userIsMultiFacilityAdmin"
          :to="{ name: 'AllFacilitiesPage' }"
          :text="coreString('allFacilitiesLabel')"
          icon="back"
        />
      </p>
      <h1>{{ coreString('classesLabel') }}</h1>
      <p>{{ $tr('classPageSubheader') }}</p>

      <p v-if="classList.length === 0">
        <KExternalLink
          v-if="isAdmin && createClassUrl"
          :text="$tr('noClassesDetailsForAdmin')"
          :href="createClassUrl"
        />
        <span v-else>
          {{ emptyStateDetails }}
        </span>
      </p>

      <CoreTable v-else>
        <template #headers>
          <th>{{ coreString('classNameLabel') }}</th>
          <th>{{ coreString('coachesLabel') }}</th>
          <th>{{ coreString('learnersLabel') }}</th>
        </template>
        <template #tbody>
          <transition-group tag="tbody" name="list">
            <tr v-for="classObj in classList" :key="classObj.id">
              <td>
                <KRouterLink
                  :text="classObj.name"
                  :to="$router.getRoute('HomePage', { classId: classObj.id })"
                  icon="classes"
                />
              </td>
              <td>
                <TruncatedItemList :items="classObj.coaches.map(c => c.full_name)" />
              </td>
              <td>
                {{ coachString('integer', { value: classObj.learner_count }) }}
              </td>
            </tr>
          </transition-group>
        </template>
      </CoreTable>
    </KPageContainer>

  </CoreBase>

</template>


<script>

  import { mapGetters, mapState } from 'vuex';
  import find from 'lodash/find';
  import commonCoreStrings from 'kolibri.coreVue.mixins.commonCoreStrings';
  import urls from 'kolibri.urls';
  import commonCoach from './common';

  export default {
    name: 'CoachClassListPage',
    mixins: [commonCoach, commonCoreStrings],
    computed: {
      ...mapGetters(['isAdmin', 'isClassCoach', 'isFacilityCoach', 'userIsMultiFacilityAdmin']),
      ...mapState(['classList']),
      // Message that shows up when state.classList is empty
      emptyStateDetails() {
        if (this.isClassCoach) {
          return this.$tr('noAssignedClassesDetails');
        }
        if (this.isAdmin) {
          return this.$tr('noClassesDetailsForAdmin');
        }
        if (this.isFacilityCoach) {
          return this.$tr('noClassesDetailsForFacilityCoach');
        }

        return '';
      },
      createClassUrl() {
        const facilityUrl = urls['kolibri:kolibri.plugins.facility:facility_management'];
        if (facilityUrl) {
          if (this.userIsMultiFacilityAdmin) {
            return `${facilityUrl()}#/${this.$route.query.facility_id}/classes`;
          }
          return facilityUrl();
        }

        return '';
      },
      appBarTitle() {
        let facilityName;
        const { facility_id } = this.$route.query;
        if (facility_id) {
          const match = find(this.$store.state.core.facilities, { id: facility_id }) || {};
          facilityName = match.name;
        }
        if (facilityName) {
          return this.coachString('coachLabelWithOneName', { name: facilityName });
        } else {
          return this.coachString('coachLabel');
        }
      },
    },
    $trs: {
      classPageSubheader: 'View learner progress and class performance',
      noAssignedClassesDetails:
        'Please consult your Kolibri administrator to be assigned to a class',
      noClassesDetailsForAdmin: 'Create a class and enroll learners',
      noClassesDetailsForFacilityCoach: 'Please consult your Kolibri administrator',
    },
  };

</script>


<style lang="scss" scoped></style>
