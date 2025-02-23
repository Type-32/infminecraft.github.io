<script setup lang="ts">
import {ref, onBeforeMount} from 'vue';
import {useRoute, useRouter} from "vue-router";
import {supabase} from "@/scripts/client";
import {
    NAvatar,
    NBackTop,
    NButton,
    NDivider,
    NDrawer, NDrawerContent,
    NIcon,
    NLayout,
    NLayoutFooter,
    NLayoutHeader,
    NSkeleton
} from "naive-ui";
import {MdPreview, MdCatalog, MdEditor} from 'md-editor-v3';
import 'md-editor-v3/lib/preview.css';
import {ArrowBack} from "@vicons/ionicons5"
import {useDataFetcher} from "@/scripts/utility/dashboard/fetch";
import Copyrighter from "@/components/Copyrighter.vue";
import type {Post} from "@/scripts/types";

const scrollElement = document.documentElement // for Markdown Catalog Renderer
const $router = useRouter()
const $route = useRoute()
const $fetcher = useDataFetcher()

const post = ref<Post>();
const author = ref()
const showAnchorDrawer = ref(false)

onBeforeMount(async () => {
    if (!$route.params.slug) {
        $router.push('/'); // Redirect to home page if no slug is present in URL
        return;
    }

    try {
        const {data, error} = await supabase
            .from('posts')
            .select('*')
            .eq('slug', $route.params.slug)
            .single();

        if (error) throw error;
        if (data)
            post.value = data;
        else
            $router.push('404'); // Redirect to home page if post is not found

        author.value = await $fetcher.fetchUserProfile(post.value?.author_id)
    } catch (error: any) {
        console.error(`Error fetching post: ${error.message}`);
        $router.push('404'); // Redirect to home page if there's any error
    }
});

function formatDateToMMMddYYYY(isoTimestamp: string): string {
    // Create a new Date object from the timestamp
    const date = new Date(isoTimestamp);

    // Create an options object for formatting
    const options: any = {year: 'numeric', month: 'short', day: 'numeric'};

    // Use Intl.DateTimeFormat to format the date
    return new Intl.DateTimeFormat('en-US', options).format(date);
}
</script>

<template>
    <NLayout class="w-full min-h-screen">
        <NLayoutHeader>
            <div class="flex flex-row p-5">
                <NButton text @click="$router.push('/')">
                    <NIcon class="mr-2">
                        <ArrowBack/>
                    </NIcon>
                    Back to Menu
                </NButton>
                <div class="flex-grow"/>
                <NButton text @click="showAnchorDrawer = true">
                    Tables of Content
                </NButton>
            </div>
        </NLayoutHeader>
        <NLayoutContent class="p-10">
            <div v-if="post">
                <div class="text-5xl font-bold mb-3">{{ post.title }}</div>
                <div class="flex text-zinc-500 items-center">
                    <div class="flex items-center">
                        <NAvatar round :src="author.avatar_url" size="small" class="mr-2"/>
                        {{ author.username }},
                    </div>
                    <div class="items-center text-zinc-500 font-bold ml-1">{{ formatDateToMMMddYYYY(post.created_at) }}</div>
                </div>
                <NDivider/>
                <MdPreview :editorId="'preview-only'" :modelValue="post.content" theme="dark" class="-mt-8" />
            </div>
            <div v-else>
                <NSkeleton height="50px"/>
                <NSkeleton height="20px" class="mt-2"/>
                <NSkeleton height="20px" class="mt-2"/>
                <NDivider/>
                <NSkeleton height="100px"/>
            </div>
            <NDrawer v-model:show="showAnchorDrawer">
                <NDrawerContent closable title="Tables of Content">
                    <MdCatalog :editorId="'preview-only'" :scrollElement="scrollElement" />
                </NDrawerContent>
            </NDrawer>
        </NLayoutContent>
        <NLayoutFooter>
            <Copyrighter/>
        </NLayoutFooter>
    </NLayout>
    <NBackTop/>
</template>

<style scoped>
.md-editor-dark {
    --md-bk-color: rgba(51, 51, 51, 0) !important;
}
</style>