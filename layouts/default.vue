<template>
<v-app>
    <v-navigation-drawer
        app
        :expand-on-hover="expandOnHover"
        right>
        <v-list
            dense
            nav>
            <v-list-item
                class="px-0"
                two-line>
                <v-list-item-avatar>
                    <img src="https://randomuser.me/api/portraits/men/81.jpg">
                </v-list-item-avatar>
                <v-list-item-content>
                    <v-list-item-title>
                        안녕하세요? 환영합니다!
                    </v-list-item-title>
                    <v-list-item-subtitle>
                        fragrantobb
                    </v-list-item-subtitle>
                </v-list-item-content>
            </v-list-item>
            <v-divider />
            <v-list-item
                v-for="(navItem, navIdx) in navList"
                :key="navIdx"
                link>
                <v-list-item-content>
                    <v-list-item-title>
                        {{navItem}}
                    </v-list-item-title>
                </v-list-item-content>
            </v-list-item>
        </v-list>
    </v-navigation-drawer>

    <v-app-bar app>
        <v-toolbar-title>fragrantobb의 정적인 열린 공간</v-toolbar-title>
        <v-btn icon>
            <v-icon>mdi-heart</v-icon>
        </v-btn>
    </v-app-bar>

    <v-content>
        <v-container fluid>
            <nuxt />
        </v-container>
    </v-content>

    <v-footer app />
</v-app>
</template>

<script>
export default {
    theme: {
        dark: true,
    },
    data() {
        return {
            navList: [
                '테스트1',
                '테스트2',
                '테스트3',
            ],
            expandOnHover: true,
        };
    },
};
</script>