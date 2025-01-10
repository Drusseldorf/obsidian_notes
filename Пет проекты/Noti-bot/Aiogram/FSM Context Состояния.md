# FSM Context. Состояния

FSM - способ отслеживать состояния пользователей.
Например, если мы собираем у пользователя его личные данные поочередено спрашивая новые данные в каждом новом сообщении.
Информация постепенно сохраняется в оперативке, и в конце мы можем сохранить все данные или как то дальше с ними работать.
То есть по сути, пользователю присваивается состояния: 1-ое: Вводит имя, 2-ое: Вводит номер телефона, и тд.

```python
	# Импорты:

		from aiogram.fsm.state import StatesGroup, State
		from aiogram.fsm.context import FSMContext

	# Создаем класс, наследуемся от StatesGroup.
	# В нем указываем поля, которые представляют собой состояния:

		class Reg(StatesGroup):
			name = State()
			number = State()

	# Как с этим работать
	# Реализуем хэндлеры, которые принимают в качестве фильтра состояния пользователя.
	# первый хэндлер - является точкой входа, когда состояния еще нет. В нем мы присваиваем пользователю состояние.
# 	Установим состояние name (то есть ждем что введет имя):

		await state.set_state(Reg.name)

	# После этого срабатывает другой хэндлер, который ждем состояния Reg.name. Попадая на него, пользователь будет вводить номер телефона, и ему присвоится другое состояние. Запомним имя и присвоим следующее состояние:

		await state.update_data(name=message.text)
		await state.set_state(Reg.number)

	# Теперь сработает обработчик для Reg.number. Сохраним данные по номеру телефона и обновим состояние, получим все данные, введенные ранее и, наконец, сбросим\\отчистим состояние.
	# data содержит словарь:

		await state.update_data(number=message.text)
		data = await state.get_data()
		await state.clear()

	# Весь процесс целиком:

		@router.message(Command('reg'))
		async def reg_one(message: Message, state: FSMContext):
			await state.set_state(Reg.name)  # установили состояние name
			await message.answer('Введите имя')

		@router.message(Reg.name)  # ждем состояния name
		async def reg_two(message: Message, state: FSMContext):
			await state.update_data(name=message.text)
			await state.set_state(Reg.number)
			await message.answer('Введите номер телефона')

		@router.message(Reg.number)  # ждем состояния number
		async def reg_three(message: Message, state: FSMContext):
			await state.update_data(number=message.text)
			data = await state.get_data()
			print(data)
			await message.answer(f'Имя: {data["name"]}\\nНомер: {data["number"]}')
			await state.clear()

```