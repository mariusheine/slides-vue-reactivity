<template>
    <div class="slidev-layout default relative">
        <div class="my-auto w-full">
            <slot />
        </div>
        <div class="absolute top-2 right-4 flex items-center gap-2 text-2xl">
            <img src="https://vuejs.org/logo.svg" class="h-[60px]"/>
            Vue.js
        </div>
        <div class="absolute bottom-2 left-2 flex items-center text-xs text-gray">
            <SlideCurrentNo /> / <SlidesTotal />
        </div>
        <div class="absolute bottom-2 right-2 flex flex-col items-center text-2xl">
            <img src="https://geprog.com/_ipx/_/logo-white.svg" class="hidden dark:block h-[30px]"/>
            <img src="https://geprog.com/_ipx/_/logo-black.svg" class="dark:hidden h-[30px]"/>
            <span class="text-[8px]" >Made with <a>https://sli.dev</a></span>
        </div>
    </div>
</template>