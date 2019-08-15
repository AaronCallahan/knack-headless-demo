<template>
  <v-layout
    column
    justify-center
    align-center
  >
    <v-flex
      xs12
      sm8
      md6
    >

      <v-data-table
        v-model="selectedRecords"
        :headers="tableHeaders"
        :items="records"
        :single-select="singleSelect"
        :loading="loading"
        item-key="id"
        show-select
        class="elevation-1"
      >

        <template v-slot:top>
          <v-toolbar flat>
            <v-toolbar-title>My Data Table</v-toolbar-title>

            <v-spacer></v-spacer>

            <v-dialog v-if="selectedRecords.length > 0" v-model="dialogDelete" max-width="500px">
              <template v-slot:activator="{ on }">
                <v-btn color="error" dark class="mb-2 mr-2" v-on="on">Delete</v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="headline">Warning</span>
                </v-card-title>

                <v-card-text>
                  <v-container>
                    <p>Are you sure you want to delete {{ selectedRecords.length }} record(s)?</p>
                  </v-container>
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="onCancelRecordDelete">Cancel</v-btn>
                  <v-btn color="blue darken-1" text @click="onRecordDelete">Delete</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>

            <v-dialog v-model="dialogNewItem" max-width="500px">
              <template v-slot:activator="{ on }">
                <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="headline">New Item</span>
                </v-card-title>

                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col v-for="header in tableHeaders" :key="header.value" cols="12" sm="6" md="4">
                        <v-text-field
                          :label="header.text"
                          v-model="newItem[header.value]"
                          />
                      </v-col>
                    </v-row>
                  </v-container>
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="onCancelNewItem">Cancel</v-btn>
                  <v-btn color="blue darken-1" text @click="onSaveNewItem">Save</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-toolbar>
        </template>

        <template v-for="header in tableHeaders" :key="" v-slot:[`item.${header.value}`]="props">
          <v-edit-dialog
            :return-value.sync="props.item[header.value]"
            @save="onSave(props.item.id, header.value, props.item[header.value])"
          > {{ props.item[header.value] }}
            <template v-slot:input>
              <v-text-field
                v-model="props.item[header.value]"
                label="Edit"
                single-line
                counter
              />
            </template>
          </v-edit-dialog>
        </template>
      </v-data-table>
    </v-flex>
  </v-layout>
</template>

<script>
  import KnackApiWrapperPublic from '../../knack-api-wrapper-public'
  const knackApiWrapperPublic = new KnackApiWrapperPublic()

  export default {
    name: `MyCustomKnackPage`,
    data () {
      return {
        singleSelect: false,
        selectedRecords: [],
        objectKey:`object_1`,
        tableHeaders: [],
        records: [],
        newItem: {},
        loading: false,
        dialogDelete: false,
        dialogNewItem: false
      }
    },
    async asyncData () {

      const { data: { records } } = await knackApiWrapperPublic.getRecords(`object_1`)
      const { data: { fields } } = await knackApiWrapperPublic.getFields(`object_1`)

      const tableHeaders = fields.map(field => {

        return {
          text: field.label,
          value: field.key
        }
      })

      return {
        records,
        tableHeaders
      }
    },
    computed: {
      selectedRecordIds () {

        const recordIds = []

        for (const record of this.selectedRecords) {

          recordIds.push(record.id)
        }

        return recordIds
      }
    },
    methods: {
      async onSave (recordId, fieldKey, fieldValue) {

        this.loading = true

        await knackApiWrapperPublic.updateRecord(recordId, this.objectKey, {
          [fieldKey]: fieldValue
        })

        this.loading = false
      },
      async onRecordDelete () {

        this.loading = true

        await knackApiWrapperPublic.deleteBatchRecordsById(this.selectedRecordIds, this.objectKey)

        setTimeout(async () => {

          const { data: { records } } = await knackApiWrapperPublic.getRecords(this.objectKey)

          this.records = records
        }, 1000)

        this.selectedRecords = []
        this.loading = false
        this.dialogDelete = false
      },
      onCancelRecordDelete () {

        this.selectedRecords = []
        this.dialogDelete = false
      },
      async onSaveNewItem () {

        this.loading = true

        const { data } = await knackApiWrapperPublic.createRecord(this.objectKey, this.newItem)

        this.records.push(data)

        this.loading = false
        this.newItem = {}
        this.dialogNewItem = false
      },
      onCancelNewItem () {

        this.newItem = {}
        this.dialogNewItem = false
      }
    }
  }
</script>
