<template>
    <div>
        <div class="content-section introduction">
            <div class="feature-intro">
                <h1>Menu</h1>
                <p>Menu is a navigation / command component that supports dynamic and static positioning.</p>
            </div>
        </div>

        <div class="content-section implementation">
            <h3 class="first">Inline</h3>
            <Menu :model="items" />

            <h3>Overlay</h3>
            <Button type="button" label="Toggle" @click="toggle" />
            <Menu ref="menu" :model="items" :popup="true" />
        </div>

        <MenuDoc />
    </div>
</template>

<script>
import MenuDoc from './MenuDoc';

export default {
    data() {
        return {
            items: [
                {
                    label: 'Options',
                    items: [{
                        label: 'Update',
                        icon: 'pi pi-refresh',
                        command: () => {
                            this.$toast.add({severity:'success', summary:'Updated', detail:'Data Updated', life: 3000});
                        }
                    },
                    {
                        label: 'Delete',
                        icon: 'pi pi-times',
                        command: () => {
                            this.$toast.add({ severity: 'warn', summary: 'Delete', detail: 'Data Deleted', life: 3000});
                        }
                    }
                ]},
                {
                    label: 'Navigate',
                    items: [{
                        label: 'Vue Website',
                        icon: 'pi pi-external-link',
                        url: 'https://vuejs.org/'
                    },
                    {
                        label: 'Router',
                        icon: 'pi pi-upload',
                        to: '/fileupload'
                    }
                ]}
            ]
        }
    },
    methods: {
        toggle(event) {
            this.$refs.menu.toggle(event);
        },
        save() {
            this.$toast.add({severity: 'success', summary: 'Success', detail: 'Data Saved', life: 3000});
        }
    },
    components: {
        'MenuDoc': MenuDoc
	}
}
</script>