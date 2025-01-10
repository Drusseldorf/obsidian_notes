# CallbackQuery

Инлайн клавиатуры имеют возможность передавать колбэк.
Колбэки мы можем обработать в хэндлерах
Тут хэндлер уже принимает не сообщение message, а callback. Который впрочем содержит практически такой же набор полей, расширенный колбэк датой:

```python
	# Пример:

		from aiogram.types import CallbackQuery

		@router.callback_query(F.data == 'callback_data')
		async def foo(callback: CallbackQuery):
			await callback.message.answer('I got your callback!')

	# В примере ниже мы добавили callback.answer(). Что это?
	# Когда мы нажмем на кнопку на клавиатуре, то она сработает, но продолжит гореть. Так происходит потому что ТГ еще не видит, что мы уже получили колбэк.
	# По сути callback.answer() - это такое короткое уведомление - текст поверх чата, который сам пропадает через секунду. Если туда не передавать аргументов, то его не будет, зато кнопка не будет мигать слишком долго.
	# Можно еще так:

		callback.answer('какой то текст', show_alert=True)

	# Тогда уже будет сообщение поверх чата, где нужно уже пользователя нажать на кнопку, то есть более "навязчивое" уведомление

	# Так же можно сделать edit, чтобы сообщение не переотправлялось, а менялось последнее:

		@router.callback_query(F.data == 'callback_data')
		async def catalog(callback: CallbackQuery):

			# чтобы кнопка не мигала после нажатия
			await callback.answer()

			# ниже делаем чтобы клава менялась на новую \\ новое сообщение, без отправки нового сообщения
			await callback.message.edit_text('Пришел колбэк callback_data', reply_markup=await kb.inline_cars())

```