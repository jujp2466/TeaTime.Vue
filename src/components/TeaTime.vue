<template>
    <div>
        <b-table striped hover :items="formattedItems" :fields="fields">
            <!-- 品項欄位 -->
            <template #cell(item)="data">
                <div>{{ data.item.name }}</div>
            </template>

            <!-- 價格欄位 -->
            <template #cell(price)="data">
                <div>{{ formatCurrency(data.item.price) }}</div>
            </template>

            <!-- 數量欄位 -->
            <template #cell(quantity)="data">
                <input v-model.number="data.item.quantity"
                       type="number"
                       min="0"
                       class="form-control"
                       :class="{'is-invalid': data.item.errors.quantity}" />
                <div v-if="data.item.errors.quantity" class="invalid-feedback">數量是必填項</div>
            </template>

            <!-- 操作按鈕 -->
            <template #cell(actions)="data">
                <button @click="handleSubmit(data.item)" class="btn btn-primary">提交</button>
            </template>
        </b-table>

        <!-- 按鈕用來動態增加品項 -->
        <button @click="addItem" class="btn btn-success mt-3">增加品項</button>
    </div>

    <div>
        <b-form-select v-model="selected" :options="options" class="form-select"></b-form-select>
        <div class="mt-3">Selected: <strong>{{ selected }}</strong></div>
    </div>
</template>

<script setup lang="ts">
    import { ref, onMounted, computed, h } from 'vue';
    import type { Ref, ComputedRef } from 'vue';
    import axios from 'axios';

    // 定義 MenuItem 型別
    interface MenuItem {
        id: number;
        name: string;
        price: number;
        quantity: number;
        errors: {
            quantity: boolean;
        };
    }
    // 格式化貨幣為新台幣
    const formatCurrency = (value: number): string => {
        return new Intl.NumberFormat('zh-TW', { style: 'currency', currency: 'TWD' }).format(value);
    };

    const items: Ref<MenuItem[]> = ref([]); // 使用明確的型別
    const fields = [
        { key: 'item', label: '品項' },
        { key: 'price', label: '價格' },
        { key: 'quantity', label: '數量' },
        { key: 'actions', label: '操作' }
    ];
    const selected: Ref<string | null> = ref(null);
    const options: Ref<{ value: string | null, text: string }[]> = ref([
        { value: null, text: 'Please select an option' }
    ]);

    // 格式化品項
    const formattedItems: ComputedRef<MenuItem[]> = computed(() => {
        return items.value.map(item => ({
            ...item,
            errors: {
                quantity: false
            }
        }));
    });

    // 驗證品項
    const validateItem = (item: MenuItem): boolean => {
        let isValid = true;

        // 驗證數量是否填寫
        item.errors.quantity = item.quantity === undefined || item.quantity <= 0;

        if (item.errors.quantity) {
            isValid = false;
        }

        return isValid;
    };

    // 提交品項
    const handleSubmit = async (item: MenuItem): Promise<void> => {
        if (!validateItem(item)) {
            return;
        }
        // 提交邏輯
        try {
            await axios.post('https://localhost:7137/api/Order/SubmitOrder', {
                itemId: item.id,
                quantity: item.quantity
            });
            alert('訂單已提交');
        } catch (error) {
            console.error('Error submitting order:', error);
        }
    };

    // 新增品項
    const addItem = (): void => {
        axios.get('https://localhost:7011/api/Menu/GetMenuItems')
            .then(response => {
                const newItems: MenuItem[] = response.data.map((menuItem: MenuItem) => ({
                    ...menuItem,
                    quantity: 0,
                    errors: {
                        quantity: false
                    }
                }));
                items.value = [...items.value, ...newItems];
                options.value = response.data.map((menuItem: MenuItem) => ({
                    value: menuItem.name,
                    text: menuItem.name
                }));
                console.log("items.value:" + JSON.stringify(items.value, null, 2));
            })
            .catch(error => {
                console.error('Error fetching menu items:', error);
            });
    };

    // 元件掛載時加載資料
    onMounted(async () => {
        await axios.get('https://localhost:7011/api/Menu/GetMenuItems')
            .then(response => {
                items.value = response.data.map((menuItem: MenuItem) => ({
                    ...menuItem,
                    quantity: 0, // 初始數量設為 0
                    errors: {
                        quantity: false
                    }
                }));

                options.value = response.data.map((menuItem: MenuItem) => ({
                    value: menuItem.name,
                    text: menuItem.name
                }));

                console.log("items.value:" + JSON.stringify(items.value, null, 2));
            })
            .catch(error => {
                console.error('Error fetching menu items:', error);
            });
    });
</script>
