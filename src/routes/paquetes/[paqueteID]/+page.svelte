<script>
	import { fly } from 'svelte/transition';
	import { slide } from 'svelte/transition';
	import { Spanish } from 'flatpickr/dist/l10n/es.js';
	import {
		StructuredList,
		StructuredListBody,
		StructuredListRow,
		StructuredListCell,
		Tile,
		Grid,
		Row,
		ComboBox,
		Column,
		Button,
		ButtonSet,
		Modal,
		ModalHeader,
		ModalBody,
		ModalFooter,
		TextInput,
		NumberInput,
		Select,
		SelectItem,
		DatePicker,
		DatePickerInput,
		TextArea,
		FormGroup,
		InlineLoading,
		ComposedModal,
		ToastNotification,
		ProgressBar,
		Accordion,
		AccordionItem
	} from 'carbon-components-svelte';

	import { enhance } from '$app/forms';

	// Logo en base64 preprocesado
	import { logob64 } from '$lib/logob64';

	import { Edit, TrashCan, Printer } from 'carbon-icons-svelte';

	const pdfFonts = {
		// download default Roboto font from cdnjs.com
		Roboto: {
			normal:
				'https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.2.7/fonts/Roboto/Roboto-Regular.ttf',
			bold: 'https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.2.7/fonts/Roboto/Roboto-Medium.ttf',
			italics:
				'https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.2.7/fonts/Roboto/Roboto-Italic.ttf',
			bolditalics:
				'https://cdnjs.cloudflare.com/ajax/libs/pdfmake/0.2.7/fonts/Roboto/Roboto-MediumItalic.ttf'
		}
	};

	import pdfMake from 'pdfmake/build/pdfmake';

	export let data;

	let open = false;
	let open2 = false;
	let creating = false;

	let precio = data.paquetes.precio;

	//fechas fix wtf
	const fecha = new Date(data.paquetes.fechasalida);

	const opciones = {
		day: '2-digit',
		month: 'short',
		year: 'numeric',
		timeZone: 'UTC' // Establece la zona horaria a UTC
	};

	const formatoFecha = new Intl.DateTimeFormat('es-ES', opciones).format(fecha);

	console.log(formatoFecha);

	const items = [
		{
			name: 'Destino',
			icon: 'bx-map',
			value: data.paquetes.nombre
		},
		{
			name: 'Precio',
			icon: 'bx-dollar',
			value: precio
		},
		{
			name: 'Salida',
			icon: 'bx-calendar-heart',
			value: new Date(data.paquetes.fechasalida).toLocaleDateString('es-ES', {
				day: '2-digit',
				month: 'short',
				year: 'numeric',
				timeZone: 'UTC'
			})
		},
		{
			name: 'Llegada',
			icon: 'bx-calendar-check',
			value: new Date(data.paquetes.fecharetorno).toLocaleDateString('es-ES', {
				day: '2-digit',
				month: 'short',
				year: 'numeric',
				timeZone: 'UTC'
			})
		},
		{
			name: 'Dias',
			icon: 'bx-sun',
			value: data.paquetes.cant_dias
		},
		{
			name: 'Noches',
			icon: 'bx-moon',
			value: data.paquetes.cant_noches
		},
		{
			name: 'Regimen',
			icon: 'bx-food-menu',
			value: data.paquetes.regimen
		},
		{
			name: 'Hotel',
			icon: 'bx-bed',
			value: data.paquetes.hotel
		},
		{
			name: 'Estado',
			icon: 'bx-check-circle',
			value: data.paquetes.estado
		},
		{
			name: 'Creado',
			icon: 'bx-calendar',
			value: new Date(data.paquetes.created).toLocaleDateString('es-ES', {
				day: '2-digit',
				month: 'short',
				year: 'numeric',
				timeZone: 'UTC'
			})
		},
		{
			name: 'Modificado',
			icon: 'bx-calendar-edit',
			value: new Date(data.paquetes.updated).toLocaleDateString('es-ES', {
				day: '2-digit',
				month: 'short',
				year: 'numeric',
				timeZone: 'UTC'
			})
		},
		{
			name: 'Observaciones',
			icon: 'bx-notepad',
			value: data.paquetes.descripcion
		}
	];

	//VALIDACIONES
	$: isValidNombre = true;
	$: validartionMessageNombre = '';
	$: isFormValid = isValidNombre ? true : false;
	$: creating = false;

	let nombre = data.paquetes.nombre;
	let cant_dias = data.paquetes.cant_dias;
	let cant_noches = data.paquetes.cant_noches;
	let hotel = data.paquetes.hotel;
	let regimen = data.paquetes.regimen;
	let estado = data.paquetes.estado;
	let fechasalida = data.paquetes.fechasalida;
	let fecharetorno = data.paquetes.fecharetorno;
	let observaciones = data.paquetes.descripcion;
	let pais_destino = data.paquetes.pais_destino;

	let toast = false;

	/**
	 * @type {HTMLFormElement}
	 */
	let form;

	/**
	 * @type {HTMLFormElement}
	 */
	let deleteForm;

	const validateNombre = () => {
		if (nombre.length < 3) {
			isValidNombre = false;
			validartionMessageNombre = 'El destino debe contener al menos 3 letras.';
		} else {
			isValidNombre = true;
			validartionMessageNombre = '';
		}
	};

	const closeModals = () => {
		open = false;
		toast = false;
		window.location.reload();
	};

	const docDefinition = {
		pageMargins: [40, 40],
		pageSize: 'A4',
		content: [
			{
				columns: [
					[
						{
							image: `${logob64}`,
							width: 72, // Ajusta el ancho de la imagen según tus necesidades
							alignment: 'left',
							margin: [0, 0]
						},
						{
							text: [
								{ text: 'Wachnitz Reisen.\n', fontSize: 14, bold: true },
								{ text: 'Empresa de viajes y turismo \n', fontSize: 10, bold: true },
								{ text: 'Legajo número: ', bold: true },
								'18376. \n',
								{ text: 'Dirección: ', bold: true },
								'La Rioja 2203 - Posadas (3300) - Misiones. \n',
								{ text: 'Teléfonos: ', bold: true },
								'+54 (3764) 222333 / +54 (3764) 424450'
							],
							fontSize: 10,
							alignment: 'left',
							width: 320,
							margin: [0, 0]
						}
					],
					{
						text: `ID PAQUETE: ${data.paquetes.id.toString().padStart(4, '0').toUpperCase()}`,
						fontSize: 10,
						alignment: 'right',
						margin: [0, 10]
					}
				]
			},

			{
				text: `MANIFIESTO DE PASAJEROS Y TRIPULACIÓN`,
				style: 'header'
			},

			{
				table: {
					headerRows: 1,
					widths: ['*', '*'],
					body: [
						[
							{
								table: {
									headerRows: 1,
									widths: ['*', '*'],
									body: [
										[
											{ text: 'DATOS CONDUCTOR', style: 'tableHeader' },
											{ text: 'DNI', style: 'tableHeader' }
										],
										[`${data.paquetes.chofer1nombre}`, `${data.paquetes.chofer1dni}`],
										[`${data.paquetes.chofer2nombre}`, `${data.paquetes.chofer2dni}`]
									]
								}
							},
							{
								table: {
									headerRows: 1,
									widths: ['*', '*'],
									body: [
										[
											{ text: 'DOMINIO VEHICULO', style: 'tableHeader' },
											{ text: 'INTERNO VEHICULO', style: 'tableHeader' }
										],

										[`${data.paquetes.dominio}`, `${data.paquetes.interno}`]
									]
								}
							}
						]
					]
				}
			},

			{
				table: {
					headerRows: 1,
					widths: ['auto', '*', 'auto', 'auto', 'auto', 'auto', 'auto', 'auto'],
					body: [
						// Encabezado de la tabla
						[
							{ text: 'NRO.', style: 'tableHeader' },
							{ text: 'APELLIDO Y NOMBRE', style: 'tableHeader' },
							{ text: 'NACIMIENTO', style: 'tableHeader' },
							{ text: 'NACIONALIDAD', style: 'tableHeader' },
							{ text: 'OCUPACIÓN', style: 'tableHeader' },
							{ text: 'SEXO', style: 'tableHeader' },
							{ text: 'DNI', style: 'tableHeader' },
							{ text: 'PAIS RESIDENCIA', style: 'tableHeader' }
						],
						// Datos de los pasajeros
						...data.ventas.reduce((rows, venta) => {
							// Agregar cada pasajero como una fila en la tabla
							venta.pasajeros.forEach((pasajero) => {
								rows.push([
									{ text: rows.length + 1, fontSize: 10 }, // Tamaño de fuente de la celda
									{ text: pasajero.apellido + ' ' + pasajero.nombre || '', fontSize: 10 },
									{
										text:
											new Date(pasajero.fechanacimiento)
												.toLocaleDateString('es-ES', {
													day: '2-digit',
													month: '2-digit',
													year: 'numeric'
												})
												.toUpperCase() || '',
										fontSize: 10
									},
									{ text: pasajero.nacionalidad || '', fontSize: 10 },
									{ text: pasajero.ocupacion || '', fontSize: 10 },
									{
										text:
											pasajero.sexo === 'MASCULINO'
												? 'M'
												: (pasajero.sexo === 'FEMENINO' ? 'F' : '') || '',
										fontSize: 10
									},
									{ text: pasajero.dni || '', fontSize: 10 },
									{ text: pasajero.nacionalidad || '', fontSize: 10 }
								]);
							});
							return rows;
						}, [])
					]
				}
			}

			// ...
		],
		styles: {
			tableFont: {
				fontSize: 8 // Ajusta el tamaño de fuente deseado para la tabla
			},

			header: {
				fontSize: 14,
				bold: true,
				alignment: 'center',
				margin: [0, 15, 0, 5]
			},
			// Estilo para el encabezado de la tabla
			tableHeader: {
				bold: true,
				fontSize: 8,
				alignment: 'center',
				fillColor: '#eeeeee' // Color de fondo del encabezado de la tabla
			},
			paragraph: {
				fontSize: 10,
				alignment: 'justify',
				margin: [0, 2, 0, 2]
			},
			bold: {
				bold: true,
				margin: [0, 5, 0, 5]
			},
			list: {
				fontSize: 8,
				margin: [0, 5, 0, 5]
			},
			page: {
				fontSize: 8,
				alignment: 'right'
			}
		}
	};

	const totaldepelotuditos = data.ventas.reduce(
		(/** @type {any} */ acc, /** @type {{ pasajeros: string | any[]; }} */ venta) =>
			acc + venta.pasajeros.length,
		0
	);
</script>

<form
	id="deleteForm"
	bind:this={deleteForm}
	action="?/delete"
	method="post"
	class="hidden"
	use:enhance
/>

<Modal
	danger
	bind:open={open2}
	modalHeading="Eliminar paquete"
	primaryButtonText="Eliminar"
	secondaryButtonText="Cancelar"
	on:click:button--secondary={() => (open2 = false)}
	on:click:button--primary={() => deleteForm.requestSubmit()}
	on:open
	on:close
	on:submit
>
	<h5>¿Está seguro que desea eliminar el paquete de {data.paquetes.nombre}?</h5>
	<p class="mt-4">
		Esta es una accion permanente y los registros asociados al paquete tambien seran eliminados
	</p>
</Modal>

<Grid>
	<Row class="justify-between p-4">
		<h1>Paquete: {data.paquetes.nombre}</h1>
		<ButtonSet class="mr-36 p-4">
			<Button size="small" on:click={() => (open2 = true)} icon={TrashCan} kind="danger"
				>Eliminar</Button
			>
			<Button size="small" icon={Edit} on:click={() => (open = true)}>Editar</Button>
			<Button
				size="small"
				kind="tertiary"
				icon={Printer}
				on:click={() => {
					pdfMake.createPdf(docDefinition, null, pdfFonts).open();
				}}>Imprimir Manifiesto</Button
			>
		</ButtonSet>
	</Row>
	<Row>
		<Column
			><Tile>
				<h2 class="">Información del paquete</h2>

				choferes y dominio {data.paquetes.chofer1nombre}
				{data.paquetes.chofer1dni}
				{data.paquetes.chofer2nombre}
				{data.paquetes.chofer2dni}
				{data.paquetes.dominio}

				<ul class="mt-4">
					{#each items as item}
						<li class="flex space-x-3 space-y-2 justify-between">
							<div>
								<!-- Icon -->
								<i class="bx text-blue-600 {item.icon}" />
								<!-- Name -->
								<span>{item.name}:</span>
							</div>

							<!-- Text -->
							<span>{item.value}</span>
						</li>
					{/each}
				</ul>
			</Tile></Column
		>
		<Column>
			<Tile>
				<h2 class="">Pasajeros {totaldepelotuditos}</h2>

				<div class="mb-4">
					<ProgressBar
						max={30}
						status={totaldepelotuditos > 30 ? 'finished' : undefined}
						labelText="Total de pasajeros:"
						helperText="{totaldepelotuditos} de {30} necesarios"
						value={totaldepelotuditos}
					/>
				</div>
				<Accordion>
					{#each data.ventas as venta}
						<AccordionItem>
							<svelte:fragment slot="title">
								<h5>
									<a class="" href="/ventas/{venta.id}"><strong>CONTRATO:</strong> {venta.id}</a>
								</h5>
								<div>
									<strong>Titular:</strong>
									{venta.cliente.nombre}
									{venta.cliente.apellido}
								</div>
							</svelte:fragment>
							<StructuredList>
								<StructuredListBody>
									{#each venta.pasajeros as pasajero}
										<StructuredListRow>
											<StructuredListCell>DNI: {pasajero.dni}</StructuredListCell>
											<StructuredListCell>{pasajero.nombre} {pasajero.apellido}</StructuredListCell>
										</StructuredListRow>
									{/each}
								</StructuredListBody>
							</StructuredList>
						</AccordionItem>
					{/each}
				</Accordion>
			</Tile></Column
		>
	</Row>
</Grid>

<ComposedModal class="" bind:open on:close={() => closeModals()}>
	<ModalHeader label="" title="Editar paquete" />
	<ModalBody hasForm hasScrollingContent>
		{#if creating}
			<div in:fly={{ y: 100 }} out:slide>
				<InlineLoading description="Guardando paquete..." />
			</div>
		{:else if toast}
			<div in:fly={{ y: 100 }} out:slide>
				<ToastNotification
					lowContrast
					kind="success"
					title="Operación exitosa"
					subtitle={new Date().toLocaleString()}
					caption="El paquete se ha actualizado correctamente."
					fullWidth
					on:close={() => (toast = false)}
				/>
			</div>
		{:else}
			<form
				id="modalForm"
				on:submit={() => {
					open = true;
					isFormValid = false;
					console.log('submit');
				}}
				method="post"
				action="?/update"
				bind:this={form}
				use:enhance={() => {
					creating = true;
					return async ({ update }) => {
						await update();
						creating = false;
						toast = true;
						isValidNombre = true;
					};
				}}
			>
				<div class="flex">
					<div class="w-[50%] flex-grow p-4">
						<!-- Contenido de la primera columna -->
						<FormGroup legendText="Destino">
							<TextInput
								id="nombre"
								bind:value={nombre}
								name="nombre"
								on:keyup={validateNombre}
								invalid={!isValidNombre}
								invalidText={validartionMessageNombre}
								labelText=""
								placeholder="Ingrese el destino"
							/>
						</FormGroup>

						<FormGroup legendText="Precio">
							<NumberInput
								id="precio"
								min={1}
								name="precio"
								invalid={precio < 0}
								invalidText="El precio debe ser mayor a 0"
								placeholder="Ingrese el precio"
								bind:value={precio}
							/>
						</FormGroup>

						<FormGroup legendText="Cantidad de días">
							<NumberInput
								id="cant_dias"
								min={1}
								name="cant_dias"
								placeholder="Ingrese la cantidad de días"
								bind:value={cant_dias}
								invalidText="La cantidad de días debe ser mayor a 0"
							/>
						</FormGroup>

						<FormGroup legendText="Cantidad de noches">
							<NumberInput
								id="cant_noches"
								min={1}
								name="cant_noches"
								placeholder="Ingrese la cantidad de noches"
								bind:value={cant_noches}
								invalidText="La cantidad de noches debe ser mayor a 0"
							/>
						</FormGroup>
					</div>
					<div class="w-[50%] flex-grow p-4">
						<!-- Contenido de la segunda columna -->

						<FormGroup legendText="Fechas salida y retorno">
							<DatePicker
								datePickerType="range"
								dateFormat="d/m/Y"
								locale={Spanish}
								bind:valueFrom={fechasalida}
								bind:valueTo={fecharetorno}
								on:change
							>
								<DatePickerInput placeholder="dd/mm/aaa" name="fechasalida" />
								<DatePickerInput placeholder="dd/mm/aaa" name="fecharetorno" />
							</DatePicker>
						</FormGroup>

						<FormGroup legendText="Pais de destino">
							<Select id="select-1" hideLabel name="pais_destino" selected={pais_destino}>
								<SelectItem disabled hidden value="Argentina" text="Seleccione un país" />
								{#each data.nacionalidades as nacionalidad}
									<SelectItem
										value={nacionalidad.destpais}
										text="{nacionalidad.destpais} {nacionalidad.codpais}"
									/>
								{/each}
							</Select>
						</FormGroup>

						<FormGroup legendText="Regimen">
							<Select id="select-1" hideLabel name="regimen" selected={regimen}>
								<SelectItem disabled hidden value="PENSION COMPLETA" text="Seleccione un regimen" />

								<SelectItem value="PENSION COMPLETA" text="PENSION COMPLETA" />
								<SelectItem value="MEDIA PENSION" text="MEDIA PENSION" />
								<SelectItem value="DESAYUNO" text="DESAYUNO" />
							</Select>
						</FormGroup>

						<FormGroup legendText="Estado">
							<Select id="select-1" hideLabel name="estado" selected={estado}>
								<SelectItem disabled hidden value="DISPONIBLE" text="Seleccione un estado" />
								<SelectItem value="NO DISPONIBLE" text="NO DISPONIBLE" />
								<SelectItem value="DISPONIBLE" text="DISPONIBLE" />
							</Select>
						</FormGroup>
					</div>
				</div>
				<!-- obervaciones -->

				<FormGroup legendText="Hotel">
					<TextInput id="hotel" name="hotel" placeholder="Ingrese el hotel" bind:value={hotel} />
				</FormGroup>

				<FormGroup legendText="Chofer 1">
					<TextInput
						id="chofer1nombre"
						name="chofer1nombre"
						placeholder="Ingrese el nombre del chofer 1"
					/>
					<NumberInput
						id="chofer1dni"
						min={1}
						name="chofer1dni"
						placeholder="Ingrese el dni del chofer 1"
					/>
				</FormGroup>

				<FormGroup legendText="Chofer 2">
					<TextInput
						id="chofer2nombre"
						name="chofer2nombre"
						placeholder="Ingrese el nombre del chofer 2"
					/>
					<NumberInput
						id="chofer2dni"
						min={1}
						name="chofer2dni"
						placeholder="Ingrese el dni del chofer 2"
					/>
				</FormGroup>

				<FormGroup legendText="Dominio del bus">
					<TextInput id="interno" name="interno" placeholder="Ingrese el interno del bus" />
					<TextInput id="dominio" name="dominio" placeholder="Ingrese el dominio del bus" />
				</FormGroup>

				<TextArea
					class="p-4"
					name="observaciones"
					labelText="Observaciones"
					placeholder="Puede ingresar observaciones si lo desea..."
					maxCount={300}
					bind:value={observaciones}
				/>
			</form>
		{/if}
	</ModalBody>
	<ModalFooter>
		<Button kind="secondary" size="lg" on:click={() => (open = false)}>Cancelar</Button>
		<Button
			id="modalSubmitButton"
			size="lg"
			type="submit"
			disabled={!isFormValid || creating}
			on:click={() => form.requestSubmit()}>Actualizar</Button
		>
	</ModalFooter>
</ComposedModal>
