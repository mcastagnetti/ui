<template>
  <UPage v-if="page">
    <UPageHeader :title="page.title" :description="page.description" :links="page.links" :headline="headline" />

    <UPageBody prose>
      <ContentRenderer v-if="page.body" :value="page" />

      <hr v-if="surround?.length">

      <UDocsSurround :surround="surround" />
    </UPageBody>

    <template v-if="page?.body?.toc?.links?.length" #right>
      <UDocsToc :links="page.body.toc.links">
        <template #bottom>
          <div class="hidden lg:block space-y-6 !mt-6">
            <UDivider v-if="page.body?.toc?.links?.length" type="dashed" />

            <UPageLinks title="Community" :links="communityLinks" />

            <UDivider type="dashed" />

            <UPageLinks title="Resources" :links="resourcesLinks" />

            <UDivider type="dashed" />

            <div class="space-y-3">
              <AdsPro />
              <AdsCarbon />
            </div>
          </div>
        </template>
      </UDocsToc>
    </template>
  </UPage>
</template>

<script setup lang="ts">
import { withoutTrailingSlash } from 'ufo'

const route = useRoute()
const { branch } = useContentSource()

definePageMeta({
  layout: 'docs'
})

const { data: page } = await useAsyncData(route.path, () => queryContent(route.path).findOne())
if (!page.value) {
  throw createError({ statusCode: 404, statusMessage: 'Page not found', fatal: true })
}

const { data: surround } = await useAsyncData(`${route.path}-surround`, () => {
  return queryContent()
    .where({
      _extension: 'md',
      _path: {
        [branch.value?.name === 'dev' ? '$eq' : '$ne']: new RegExp('^/dev')
      },
      navigation: {
        $ne: false
      }
    })
    .only(['title', 'description', '_path'])
    .findSurround(withoutTrailingSlash(route.path))
})

const headline = computed(() => findPageHeadline(page.value))

useSeoMeta({
  titleTemplate: '%s - Nuxt UI',
  title: page.value.title,
  ogTitle: `${page.value.title} - Nuxt UI`,
  description: page.value.description,
  ogDescription: page.value.description
})

defineOgImage({
  component: 'Docs',
  title: page.value.title,
  description: page.value.description,
  headline: headline.value
})

const communityLinks = computed(() => [{
  icon: 'i-heroicons-pencil-square',
  label: 'Edit this page',
  to: `https://github.com/nuxt/ui/edit/dev/docs/content/${branch.value?.name === 'dev' ? page?.value?._file.split('/').slice(1).join('/') : page?.value?._file}`,
  target: '_blank'
}, {
  icon: 'i-heroicons-star',
  label: 'Star on GitHub',
  to: 'https://github.com/nuxt/ui',
  target: '_blank'
}, {
  icon: 'i-heroicons-lifebuoy',
  label: 'Contributing',
  to: '/getting-started/contributing'
}, {
  label: 'Roadmap',
  icon: 'i-heroicons-map',
  to: '/roadmap'
}])

const resourcesLinks = [{
  icon: 'i-simple-icons-figma',
  label: 'Figma Kit',
  to: 'https://www.figma.com/community/file/1288455405058138934',
  target: '_blank'
}, {
  label: 'Playground',
  icon: 'i-simple-icons-stackblitz',
  to: 'https://stackblitz.com/edit/nuxt-ui',
  target: '_blank'
}, {
  icon: 'i-simple-icons-nuxtdotjs',
  label: 'Nuxt docs',
  to: 'https://nuxt.com',
  target: '_blank'
}]
</script>
