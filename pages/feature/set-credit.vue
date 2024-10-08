<script lang="ts" setup>
import { IconRefresh } from '@arco-design/web-vue/es/icon';

import { nanoid } from 'nanoid';

import { Bot } from '@starlight-dev-team/fanbook-api-sdk';
import type {
  GuildCredit,
} from '@starlight-dev-team/fanbook-api-sdk/dist/types';

import { useAccountStore } from '~~/stores/account';

import { tryBigintify } from '~~/utils/util';

import { BotErrorCode } from '~/utils/bot';

import {
  Button,
  Form,
  FormItem,
  Input,
  Message,
  TypographyTitle,
  Spin,
} from '@arco-design/web-vue';
import type { FieldRule } from '@arco-design/web-vue';

definePageMeta({
  title: '设置荣誉卡槽',
  requiredAuth: true,
});

interface Input {
  guild?: bigint;
  user?: bigint;
  credit: GuildCredit;
}
const input = reactive({
  credit: {
    id: '',
    authority: {
      icon: '',
      name: '',
    },
    slots: [[{
      image: '',
      value: '',
    }]],
    title: {
      icon: '',
    },
  },
} as Input);

const REQUEIRE_RULE: FieldRule = {
  required: true,
  message: '本项必填',
};

type Status = 'default' | 'loading';
const status = ref('default' as Status);

const bot = new Bot(useAccountStore().activeBotToken as string);

function generateId() {
  input.credit.id = nanoid();
}

function bigintValidator(value: string, cb: (error?: string) => void) {
  if (tryBigintify(value) === undefined) cb('数据填写错误');
  else cb(undefined);
}

async function onSubmit() {
  status.value = 'loading';
  try {
    await bot.setGuildUserCredit({
      guild: input.guild,
      user: input.user as bigint,
      credit: input.credit,
    });
    Message.success({
      content: '设置成功',
      duration: 2500,
    });
  } catch (err) {
    console.error(err);
    //如果错误码在BotErrorCode中，则显示错误码对应的错误信息
    if (err.response?.data?.error_code in BotErrorCode) {
      Message.error({
        content: '设置失败：'+BotErrorCode[err.response?.data?.error_code],
        duration: 6000,
      });
    } else {
    Message.error({
      content: '设置失败：'+err.response?.data?.description,
      duration: 6000,
    });
  }}
  status.value = 'default';
}
</script>

<template>
  <Spin class='form-wrapper' :loading='status === "loading"' tip='正在执行'>
    <Form
      class='form'
      :model='input'
      :disabled='status === "loading"'
      auto-label-width
      @submit-success='onSubmit'
    >
      <GuildInputForm
        v-model='input.guild'
        field='guild'
        required
      />
      <UserInputForm
        v-model='input.user'
        :guild='input.guild'
        field='user'
        required
      />
      <FormItem
        label='自定义 ID'
        field='credit.id'
        tooltip='荣誉卡槽唯一 ID ，修改/删除时需要此 ID'
        :rules='[REQUEIRE_RULE, { minLength: 10, message: "至少10字符" }]'
      >
        <Input v-model='input.credit.id'>
          <template #append>
            <Button type='text' @click='generateId'>
              <IconRefresh />
            </Button>
          </template>
        </Input>
      </FormItem>
      <TypographyTitle :heading='4'>标题栏配置</TypographyTitle>
      <FormItem
        label='标题栏图片链接'
        field='credit.authority.icon'
        :rules='REQUEIRE_RULE'
      >
        <Input v-model='input.credit.authority.icon' />
      </FormItem>
      <FormItem
        label='标题栏文字'
        field='credit.authority.name'
        :rules='REQUEIRE_RULE'
      >
        <Input v-model='input.credit.authority.name' />
      </FormItem>
      <TypographyTitle :heading='4'>插槽配置</TypographyTitle>
      <FormItem
        label='插槽图片链接'
        field='credit.slots[0][0].image'
      >
        <Input v-model='input.credit.slots[0][0].image' />
      </FormItem>
      <FormItem
        label='插槽图片描述'
        field='credit.slots[0][0].value'
      >
        <Input v-model='input.credit.slots[0][0].value' />
      </FormItem>
      <FormItem
        label='勋章图片链接'
        field='credit.title.icon'
        tooltip='显示在昵称左侧的小图'
      >
        <Input v-model='input.credit.title.icon' />
      </FormItem>
      <FormItem class='operations'>
        <Button type='primary' html-type='submit'>
          设置荣誉卡槽
        </Button>
      </FormItem>
    </Form>
  </Spin>
</template>

<style scoped>
.form-wrapper {
  display: block;
  width: 70%;
  margin: 0 auto;
}
body.mobile .form-wrapper {
  width: 90vw;
}
.form {
  width: 100%;
}
h4 {
  margin-top: 0;
  padding-bottom: 2px;
  border-bottom: 1px solid var(--color-text-4);
  text-align: center;
}
.operations {
  margin-top: 4px;
}
</style>
