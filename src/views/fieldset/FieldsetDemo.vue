<template>
    <div>
        <div class="content-section introduction">
            <div class="feature-intro">
                <h1>Fieldset</h1>
                <p>Fieldset is a grouping component with the optional content toggle feature.</p>
            </div>
        </div>

        <div class="content-section implementation">
            <h3 class="first">Regular</h3>
            <Fieldset legend="Godfather I">
                The story begins as Don Vito Corleone, the head of a New York Mafia family, oversees his daughter's wedding.
                His beloved son Michael has just come home from the war, but does not intend to become part of his father's business.
                Through Michael's life the nature of the family business becomes clear. The business of the family is just like the head of the family,
                kind and benevolent to those who give respect, but given to ruthless violence whenever anything stands against the good of the family.
            </Fieldset>

            <h3>Toggleable</h3>
            <Fieldset legend="Godfather I" :toggleable="true">
                The story begins as Don Vito Corleone, the head of a New York Mafia family, oversees his daughter's wedding.
                His beloved son Michael has just come home from the war, but does not intend to become part of his father's business.
                Through Michael's life the nature of the family business becomes clear. The business of the family is just like the head of the family,
                kind and benevolent to those who give respect, but given to ruthless violence whenever anything stands against the good of the family.
            </Fieldset>
        </div>

        <FieldsetDoc/>
    </div>
</template>

<script>
import FieldsetDoc from './FieldsetDoc';

export default {
    data() {
        return {
            value1: '',
            value2: '',
            value3: 'PrimeVue'
        }
    },
    components: {
        'FieldsetDoc': FieldsetDoc
    }
}
</script>
