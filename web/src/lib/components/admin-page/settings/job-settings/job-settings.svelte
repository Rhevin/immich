<script lang="ts">
  import { getJobName } from '$lib/utils';
  import { JobName, type SystemConfigDto, type SystemConfigJobDto } from '@immich/sdk';
  import { isEqual } from 'lodash-es';
  import { createEventDispatcher } from 'svelte';
  import { fade } from 'svelte/transition';
  import type { SettingsEventType } from '../admin-settings';
  import SettingButtonsRow from '../setting-buttons-row.svelte';
  import SettingInputField, { SettingInputFieldType } from '../setting-input-field.svelte';

  export let savedConfig: SystemConfigDto;
  export let defaultConfig: SystemConfigDto;
  export let config: SystemConfigDto; // this is the config that is being edited
  export let disabled = false;

  const dispatch = createEventDispatcher<SettingsEventType>();

  const jobNames = [
    JobName.ThumbnailGeneration,
    JobName.MetadataExtraction,
    JobName.Library,
    JobName.Sidecar,
    JobName.SmartSearch,
    JobName.FaceDetection,
    JobName.FacialRecognition,
    JobName.VideoConversion,
    JobName.StorageTemplateMigration,
    JobName.Migration,
  ];

  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  function isSystemConfigJobDto(jobName: any): jobName is keyof SystemConfigJobDto {
    return jobName in config.job;
  }
</script>

<div>
  <div in:fade={{ duration: 500 }}>
    <form autocomplete="off" on:submit|preventDefault>
      {#each jobNames as jobName}
        <div class="ml-4 mt-4 flex flex-col gap-4">
          {#if isSystemConfigJobDto(jobName)}
            <SettingInputField
              inputType={SettingInputFieldType.NUMBER}
              {disabled}
              label="{getJobName(jobName)} Concurrency"
              desc=""
              bind:value={config.job[jobName].concurrency}
              required={true}
              isEdited={!(config.job[jobName].concurrency == savedConfig.job[jobName].concurrency)}
            />
          {:else}
            <SettingInputField
              inputType={SettingInputFieldType.NUMBER}
              label="{getJobName(jobName)} Concurrency"
              desc=""
              value="1"
              disabled={true}
              title="This job is not concurrency-safe."
            />
          {/if}
        </div>
      {/each}

      <div class="ml-4">
        <SettingButtonsRow
          on:reset={({ detail }) => dispatch('reset', { ...detail, configKeys: ['job'] })}
          on:save={() => dispatch('save', { job: config.job })}
          showResetToDefault={!isEqual(savedConfig.job, defaultConfig.job)}
          {disabled}
        />
      </div>
    </form>
  </div>
</div>
